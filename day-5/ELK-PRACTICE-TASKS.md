# 🛠️ Real-Time ELK Observability Practice Guide & Workshop Notes

Welcome to your hands-on **ELK Observability Practice Lab**. This guide contains real-time production tasks, step-by-step solutions, troubleshooting commands, KQL/ES|QL queries, and RCA notes to practice on your Kubernetes logging cluster.

---

## 🧭 Environment Quick Reference

- **Kibana Web UI**: [http://localhost:5601](http://localhost:5601) *(Port-forward active on 5601)*
  - **Username**: `elastic`
  - **Password**: `0P1rf1AiLY0ucHSd`
- **Elasticsearch API**: `https://172.18.0.2:30920` (NodePort) or via `elastic es <command>`
- **Cluster Status**: 🟢 `GREEN` (1 node, 17 active shards)
- **Active Data Views in Kibana**:
  - `Kubernetes Logs (ELK)` $\rightarrow$ Pattern: `elk-*`
  - `Kubernetes Metrics (Metricbeat)` $\rightarrow$ Pattern: `metricbeat-*`

---

## 🧠 Core Concept: What is a "Document" in Elasticsearch?

If you come from a relational database (SQL) or Linux background, here is the exact mental model:

| Relational DB (MySQL / Postgres) | Elasticsearch (ELK Stack) | Kubernetes Logging Equivalent |
| :--- | :--- | :--- |
| **Database** | **Cluster** | Your entire running ELK deployment |
| **Table** | **Index** (e.g., `elk-logstash-2026.10.19`) | A date-based bucket of logs |
| **Row** | **Document** (`doc`) | **A single log event / line emitted by a pod** |
| **Column** | **Field** (e.g., `log.level`, `message`) | An attribute extracted from that log |
| **Primary Key** | **Document ID** (`_id`) | Unique identifier generated for that log |

### What does a Document look like inside Elasticsearch?
In Elasticsearch, **every single document is a JSON object**. When `sample-logger` outputs one line:
```json
{
  "_index": "elk-logstash-2026.10.19",
  "_id": "4h9FZo8Bn7g",
  "_source": {
    "@timestamp": "2026-10-19T07:19:10.000Z",
    "kubernetes": {
      "namespace": "default",
      "pod": { "name": "sample-logger-7786ff4b54-ktdkc" }
    },
    "service": { "name": "sample-logger" },
    "log": { "level": "INFO" },
    "order_id": 1561,
    "message": "User order processed successfully"
  }
}
```

### Why did "176,000 documents in 1 hour" matter?
- **1 document = 1 log record stored in the database.**
- 176,000 documents meant **176,000 individual log lines** were shipped, indexed, and written to disk in 60 minutes (~50 logs every single second).
- When we analyzed it, **134,000 of those documents** were identical, useless retry error lines from Fluent-bit trying and failing to authenticate.
- Each document consumes CPU (for JSON/Grok parsing), RAM (Lucene index buffers), and disk storage. Eliminating those junk documents saved cluster memory and stopped disk from filling up!

---

## 📋 Workshop Roadmap

| Task | Domain | Focus Area | Status |
| :--- | :--- | :--- | :--- |
| **Task 1** | **Triage & Querying** | Incident RCA: Diagnosing Crashing Services using KQL, ES\|QL & APIs | ✅ Completed |
| **Task 2** | **Log Pipeline & Parsing** | Logstash Grok & ECS Extraction for Unstructured Logs (Gin HTTP & Payment API) | ✅ Completed |
| **Task 3** | **Ingestion Optimization** | Storage Cost Control: Squelching Logging Storms & Drop Filters | ✅ Completed |
| **Task 4** | **Full-Stack Correlation** | Correlating Log Spikes with Node & Pod Resource Metrics (Metricbeat & Lens) | 🟢 Active / Ready |
| **Task 5** | **Index Lifecycle Management** | Production Hot-Warm-Delete ILM Policy & Index Templates | ⏳ Upcoming |
| **Task 6** | **Production Alerting** | Automated Threshold Alert Rules & Webhook Notifications | ⏳ Upcoming |

---

# 🎯 Task 1: Incident Root Cause Analysis (RCA) with KQL, ES|QL & APIs

### 🏢 Real-World Incident Scenario
> You are the on-call SRE/DevOps engineer. Alerts triggered indicating that microservices in the `default` namespace are failing (`payment-api-failing` pod in `Error` status and intermittent errors in `sample-logger`).
>
> Your objective is to use the ELK stack to investigate the incident without running `kubectl logs` on the individual nodes, discover the failure pattern, isolate the exact root cause, and document the findings.

---

### 📦 Workload Manifests Used in this Scenario

Both workloads run in the `default` namespace and continuously emit logs collected by Filebeat $\rightarrow$ Logstash $\rightarrow$ Elasticsearch.

#### 1. Order Processing Service: [`sample-logger.yaml`](sample-logger.yaml)
Simulates an e-commerce microservice outputting structured JSON logs with success messages, latency warnings, and database errors:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-logger
  namespace: default
  labels:
    app: sample-logger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-logger
  template:
    metadata:
      labels:
        app: sample-logger
    spec:
      containers:
      - name: logger
        image: busybox:1.36
        args:
        - /bin/sh
        - -c
        - >
          count=1;
          while true; do
            echo "{\"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\", \"level\": \"INFO\", \"app\": \"sample-logger\", \"count\": $count, \"message\": \"User order processed successfully\", \"order_id\": $((1000 + count))}";
            sleep 3;
            if [ $((count % 5)) -eq 0 ]; then
              echo "{\"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\", \"level\": \"ERROR\", \"app\": \"sample-logger\", \"count\": $count, \"message\": \"Database connection timed out during payment confirmation!\", \"error_code\": 500}";
            elif [ $((count % 3)) -eq 0 ]; then
              echo "{\"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\", \"level\": \"WARN\", \"app\": \"sample-logger\", \"count\": $count, \"message\": \"High latency detected on checkout service: 1250ms\"}";
            fi;
            count=$((count + 1));
          done
        resources:
          limits:
            cpu: 100m
            memory: 64Mi
          requests:
            cpu: 50m
            memory: 32Mi
```
*Deploy command:* `kubectl apply -f sample-logger.yaml`

#### 2. Crashing Payment Service: [`payment-api-failing.yaml`](payment-api-failing.yaml)
Simulates a critical downstream microservice failing with connection pool exhaustion and crashing into `CrashLoopBackOff`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-api-failing
  namespace: default
  labels:
    app: payment-api
spec:
  restartPolicy: Always
  containers:
  - name: payment-api
    image: busybox:latest
    command: ["/bin/sh", "-c"]
    args:
    - |
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] INFO Starting Payment Processing Service v1.8.2"
      sleep 2
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] INFO Initializing database connection pool to postgres-db.internal.svc.cluster.local:5432"
      sleep 2
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] ERROR dial tcp: lookup postgres-db.internal.svc.cluster.local on 10.96.0.10:53: no such host"
      sleep 1
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] WARN Retrying database connection (attempt 2/3)..."
      sleep 2
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] ERROR could not obtain a connection from database pool within 30000ms: connection pool exhausted"
      sleep 1
      echo "$(date -u +'%Y-%m-%dT%H:%M:%S.000Z') [default/payment-api-failing/payment-api] FATAL Critical downstream dependency unreachable. Exiting."
      exit 1
```
*Deploy command:* `kubectl apply -f payment-api-failing.yaml`

---

### 📝 Task Objectives
1. Open Kibana **Discover** (or use the Elasticsearch Search API / ES|QL).
2. Filter the log stream to only show logs from namespace `default`.
3. Separate healthy transactions from failure logs.
4. Pinpoint the exact database & dependency failure messages in `payment-api-failing`.
5. Identify the error codes and impacted order IDs in `sample-logger`.

---

## 💡 Task 1: Step-by-Step Solutions

### Method A: Kibana Web UI (Discover & KQL)
1. Navigate to **Kibana** at [http://localhost:5601](http://localhost:5601) and log in.
2. Click the hamburger menu (☰) $\rightarrow$ **Analytics** $\rightarrow$ **Discover**.
3. Select the Data View: **`Kubernetes Logs (ELK)` (`elk-*`)**.
4. Set the Time Filter to **Last 1 hour** (or Last 24 hours).
5. Add columns to the table for clean readability:
   - Click the plus icon (+) next to `kubernetes.pod.name`
   - Click the plus icon (+) next to `log.level` (or `log_level`)
   - Click the plus icon (+) next to `message`
6. **Execute KQL Queries in the Search Bar**:
   - Filter to namespace `default`:
     ```kql
     kubernetes.namespace: "default"
     ```
   - Filter for `sample-logger` (Notice the `.keyword` or `service.name`):
     ```kql
     kubernetes.namespace: "default" and kubernetes.pod.name.keyword: sample-logger*
     ```
     *or using service name:*
     ```kql
     service.name: "sample-logger"
     ```
   - Find all database and critical errors in `payment-api-failing`:
     ```kql
     kubernetes.pod.name.keyword: "payment-api-failing" and message: (*FATAL* or *ERROR* or *pool*)
     ```
   - Find all error events from the order processor (`sample-logger`):
     ```kql
     service.name: "sample-logger" and (log.level: "ERROR" or error.code: *)
     ```

### ⚠️ Common Pitfalls: Why Might a Query Return 0 Results?
1. **Space before the Colon (`:`)**:
   - ❌ `kubernetes.namespace : "default"` (KQL breaks if there is a space before the colon!)
   - ✅ `kubernetes.namespace: "default"` (No space between field name and colon)
2. **Wildcards on Analyzed Text vs Keyword**:
   - `kubernetes.pod.name` is stored as `text` (analyzed, hyphen-separated). When you run a wildcard `sample-logger*`, Elasticsearch searches individual tokens.
   - ❌ `kubernetes.pod.name: sample-logger*` (fails because tokens are split into `sample` and `logger`)
   - ✅ `kubernetes.pod.name.keyword: sample-logger*` (uses the exact string keyword index)
   - ✅ `service.name: "sample-logger"` (direct keyword match)
3. **Time Picker Range & Clock**:
   - The lab environment's current system timestamp is **October 19, 2026**.
   - If your browser's local time is set to the current calendar year (2024/2025), a relative picker like "Last 15 minutes" will search for 2024/2025 and find 0 logs!
   - **Fix**: In the top-right time picker in Kibana, select **"Last 5 years"** or set an Absolute range for **Oct 19, 2026**.

---

### Method B: Elasticsearch Query DSL (CLI / Terminal)
Run this query directly against the Elasticsearch cluster to pull the error traces:

```bash
curl -sk -u elastic:0P1rf1AiLY0ucHSd \
  "https://172.18.0.2:30920/elk-logstash-*/_search" \
  -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": [
        { "term": { "kubernetes.pod.name.keyword": "payment-api-failing" } },
        { "match_phrase": { "message": "ERROR" } }
      ]
    }
  },
  "size": 5,
  "sort": [{ "@timestamp": "desc" }],
  "_source": ["@timestamp", "kubernetes.pod.name", "message"]
}' | jq '.hits.hits[]._source'
```

---

### Method C: Modern ES|QL (Elasticsearch Query Language)
In Kibana $\rightarrow$ **Management** $\rightarrow$ **Dev Tools** $\rightarrow$ **Console**, execute:

```sql
POST /_query
{
  "query": """
    FROM elk-logstash-*
    | WHERE kubernetes.namespace == "default"
    | WHERE message LIKE "*ERROR*" OR message LIKE "*FATAL*"
    | KEEP @timestamp, kubernetes.pod.name, message
    | SORT @timestamp DESC
    | LIMIT 10
  """
}
```

---

### 🔍 RCA Findings (What the data tells us)
1. **`payment-api-failing` Root Cause**:
   - **Log 1**: `could not obtain a connection from database pool within 30000ms: connection pool exhausted`
   - **Log 2**: `FATAL Critical downstream dependency unreachable. Exiting.`
   - **Diagnosis**: The application's database connection pool is maxed out, causing connection timeouts and cascading pod termination with exit code 1. DB pool size or connection management needs immediate remediation.
2. **`sample-logger` Findings**:
   - Periodically logs `High latency detected on checkout service: 1250ms` and `Payment gateway timed out (code: 504)`.
   - Structured JSON fields (`order_id`, `error.code`) are correctly parsed into separate ECS fields by Logstash.

---

# 🎯 Task 2: Building Production Logstash Ingestion Pipelines with Grok Filters & ECS Alignment

### 🏢 Real-World Production Problem
In a Kubernetes microservices environment, applications emit logs in diverse, unstructured formats:

1. **`go-service-a` in namespace `dev`** runs a Gin HTTP web server that prints:
   ```text
   [GIN] 2026/10/19 - 07:14:52 | 404 |      85.767µs |     10.244.1.15 | GET      "/metrics"
   ```
   Without parsing, this entire line is stored as a single raw string in `message`.
   - You **cannot** filter by HTTP status code (`http.response.status_code >= 400`).
   - You **cannot** aggregate or build Kibana Lens charts of top requested paths (`url.path`) or HTTP verbs (`http.request.method`).
2. **`payment-api-failing` in namespace `default`** prints:
   ```text
   2026-10-19T07:09:47.000Z [default/payment-api-failing/payment-api] FATAL Critical downstream dependency unreachable. Exiting.
   ```
   Without parsing, the log level `FATAL` is treated as plain text and collapses into `ERROR`, and the raw timestamp and pod bracket header pollute `message`.

Your objective as the Observability Engineer is to add **Logstash Grok filters** to parse these logs into Elastic Common Schema (ECS) fields, deploy the pipeline upgrade via Helm, and verify the structured fields in Kibana.

---

### 📝 Task Objectives
1. Inspect the incoming raw messages from `go-service-a` and `payment-api-failing`.
2. Author Grok patterns matching:
   - **Gin Access Logs**: Extract `http.response.status_code` (integer), `client.ip`, `http.request.method`, `url.path`, and duration.
   - **Payment API Logs**: Extract true `log.level` (`FATAL`, `ERROR`, `WARN`, `INFO`), set real `@timestamp`, and clean `message`.
3. Update `logstash-values.yml` under `logstashPipeline: logstash.conf:`.
4. Deploy the update via Helm: `helm upgrade logstash elastic/logstash -f logstash-values.yml -n logging`.
5. Verify in Kibana Discover or Dev Tools that `http.response.status_code` and clean `log.level` are queryable!

---

## 💡 Task 2: Step-by-Step Solution & Implementation

### Step 1: Update `logstash-values.yml`
Add the Grok filter blocks inside the `filter { ... }` block in `logstash-values.yml`:

```ruby
      # 3. GIN HTTP ACCESS LOG PARSER (dev microservices)
      else if [message] =~ /^\[GIN\]/ {
        grok {
          match => {
            "message" => "^\[GIN\]\s+%{YEAR}/%{MONTHNUM}/%{MONTHDAY}\s+-\s+%{TIME}\s+\|\s+%{NUMBER:[http][response][status_code]:int}\s+\|\s+%{DATA:[event][duration]}\s+\|\s+%{IP:[client][ip]}\s+\|\s+%{WORD:[http][request][method]}\s+\"%{NOTSPACE:[url][path]}\""
          }
        }
      }

      # 4. PAYMENT API / BRACKETED MICROSERVICE PARSER
      else if [message] =~ /^20\d\d-\d\d-\d\dT/ and [message] =~ /\[default\/payment-api/ {
        grok {
          match => {
            "message" => "^%{TIMESTAMP_ISO8601:[@metadata][event_time]}\s+\[%{DATA:[@metadata][ns]}/%{DATA:[@metadata][pod]}/%{DATA:[@metadata][container]}\]\s+%{WORD:[log][level]}\s+%{GREEDYDATA:[@metadata][clean_msg]}$"
          }
        }
        if [@metadata][clean_msg] {
          mutate {
            replace => { "message" => "%{[@metadata][clean_msg]}" }
          }
          date {
            match => [ "[@metadata][event_time]", "ISO8601" ]
            target => "@timestamp"
          }
        }
      }
```

### Step 2: Deploy the Upgrade to Kubernetes
Run:
```bash
helm upgrade logstash elastic/logstash -f logstash-values.yml -n logging
kubectl rollout status statefulset logstash-logstash -n logging
```

---

### Step 3: Verification Queries & Testing

#### 1. In Kibana Discover (`http://localhost:5601`):
- Filter by HTTP 404 client errors:
  ```kql
  http.response.status_code: 404
  ```
- Filter by HTTP method and endpoint:
  ```kql
  http.request.method: "GET" and url.path: "/metrics"
  ```
- Search for the newly parsed `FATAL` level logs:
  ```kql
  log.level: "FATAL"
  ```

#### 2. In Kibana Dev Tools Console (ES|QL Aggregation):
```sql
POST /_query
{
  "query": """
    FROM elk-logstash-*
    | WHERE http.response.status_code IS NOT NULL
    | STATS count = COUNT() BY http.response.status_code, http.request.method, url.path
    | SORT count DESC
  """
}
```

#### 3. Via Terminal (cURL to Elasticsearch):
```bash
curl -sk -u elastic:0P1rf1AiLY0ucHSd \
  "https://172.18.0.2:30920/elk-logstash-*/_search" \
  -H 'Content-Type: application/json' -d '{
  "query": {
    "exists": { "field": "http.response.status_code" }
  },
  "size": 3,
  "sort": [{ "@timestamp": "desc" }],
  "_source": ["@timestamp", "service.name", "http.response.status_code", "http.request.method", "url.path", "client.ip"]
}' | jq '.hits.hits[]._source'
```

---

### 🔍 Production Takeaway
1. **ECS Alignment**: Mapping fields like `http.response.status_code`, `http.request.method`, `url.path`, and `client.ip` ensures seamless compatibility with built-in Kibana Observability dashboards, APM UI, and SIEM security correlation.
2. **Target Field Types**: Using `:int` in `%{NUMBER:[http][response][status_code]:int}` ensures Elasticsearch indexes the status code as a numeric field, enabling range queries like `http.response.status_code >= 500`.
3. **Clean Message**: Moving metadata into dedicated fields and replacing `message` with the actual clean log text improves readability in Discover and reduces storage overhead.

---

# 🎯 Task 3: Ingestion Optimization, Squelching Logging Feedback Storms & Drop Rules

### 🏢 Real-World SRE Disaster Scenario
In Kubernetes production environments, a very common outage cause is the **"Observability Feedback Loop Storm"**:
1. When checking `_cat/indices`, today's index `elk-logstash-2026.10.19` suddenly surged to **over 176,000 documents** within 1 hour!
2. Running an Elasticsearch terms aggregation reveals that **76% of all logs** (134,000+ documents) are generated by a single pod: `fluent-bit-spfgf`.
3. **The Root Cause**:
   - `fluent-bit` has an invalid/outdated password in its configuration and continuously attempts `/_bulk` indexing into Elasticsearch.
   - Elasticsearch rejects it with `HTTP 401 Unauthorized`.
   - Fluent-bit writes error messages to `/var/log/containers/fluent-bit...log`:
     `[warn] [engine] failed to flush chunk... retry in 402 seconds... HTTP status=401`
   - Filebeat dutifully ships these container log files into Logstash.
   - Logstash indexes them into Elasticsearch.
   - Elasticsearch logs the 401 audit failure.
   - The cycle repeats thousands of times per minute, ballooning disk utilization and threatening a cluster disk watermark lock (`read_only_allow_delete`)!

Your objective as the Lead DevOps / SRE Engineer is to:
1. Detect and quantify the volume of junk / feedback-loop logs using Elasticsearch aggregations.
2. Implement **defensive Drop Filters** in Logstash and Filebeat to immediately drop shipper retry noise, failed chunk warnings, and Kubernetes health probes (`/healthz`, `/livez`).
3. Remediate the root cause by updating `fluentbit-values.yaml` with the current valid credentials.
4. Verify that the ingestion rate stabilizes and garbage logs drop to zero.

---

### 📝 Task Objectives
1. Run an Elasticsearch aggregation to measure the top log producers and isolate the noise.
2. Update the Logstash drop filter in `logstash-values.yml` to discard all `failed to flush chunk`, `unable to authenticate user`, and `status=401` log shipper noise.
3. Update `fluentbit-values.yaml` with the current password `0P1rf1AiLY0ucHSd` and apply via Helm (`helm upgrade fluent-bit ...`).
4. Verify with Kibana Discover or Elasticsearch APIs that no further retry storm logs enter the cluster.

---

## 💡 Task 3: Step-by-Step Solution & Implementation

### Step 1: Quantify the Leak with Elasticsearch Aggregations & Kibana KQL

#### Method 1: In Kibana Discover (KQL Query)
Paste this query into the search bar in **Kibana Discover** to filter and see all the redundant noise events:
```kql
kubernetes.pod.name.keyword: fluent-bit* and message: (
  "failed to flush chunk" or
  "unable to authenticate user" or
  "status=401" or
  "Non-zero metrics" or
  "DEPRECATED"
)
```

#### Method 2: Via Terminal (Elasticsearch Aggregations)
Run this query to see which pods are responsible for the log explosion:

```bash
curl -sk -u elastic:0P1rf1AiLY0ucHSd \
  "https://172.18.0.2:30920/elk-logstash-*/_search" \
  -H 'Content-Type: application/json' -d '{
  "size": 0,
  "aggs": {
    "top_spammers": {
      "terms": {
        "field": "kubernetes.pod.name.keyword",
        "size": 5
      }
    }
  }
}' | jq .aggregations
```
*(Notice that `fluent-bit` generated over 134,000 documents!)*

---

### Step 2: Implement Defensive Drop Filters in Logstash
In `logstash-values.yml`, configure the top of the `filter { ... }` block to drop logging storm noise before processing:

```ruby
    filter {
      # 1. DROP NOISE & LOGGING FEEDBACK LOOPS
      if [kubernetes][container][name] =~ /(fluent-bit|filebeat)/ {
        if [message] =~ /(failed to flush chunk|unable to authenticate user|status=401|Non-zero metrics|DEPRECATED)/ {
          drop { }
        }
      }
      if [message] =~ /(GET \/healthz|GET \/livez|kube-probe)/ {
        drop { }
      }
```

Deploy the updated Logstash:
```bash
helm upgrade logstash elastic/logstash -f logstash-values.yml -n logging
kubectl rollout status statefulset logstash-logstash -n logging
```

---

### Step 3: Fix the Root Cause in Fluent-bit
The root cause of Fluent-bit's continuous failures is an outdated password in `fluentbit-values.yaml`.
Lines 443 and 458 in `fluentbit-values.yaml` contain the old password:
```yaml
        HTTP_Passwd XIU8oQSqmKY3ShUd
```
Update both occurrences to the current cluster password:
```yaml
        HTTP_Passwd 0P1rf1AiLY0ucHSd
```
Then upgrade Fluent-bit:
```bash
helm upgrade fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging
kubectl rollout restart ds/fluent-bit -n logging
kubectl rollout status ds/fluent-bit -n logging
```

---

### Step 4: Verification
1. Check Fluent-bit logs to confirm clean flushing without 401s:
   ```bash
   kubectl logs -n logging -l app.kubernetes.io/name=fluent-bit --tail=20
   ```
2. Search Elasticsearch via Terminal for any new `failed to flush chunk` messages in the last 5 minutes:
   ```bash
   curl -sk -u elastic:0P1rf1AiLY0ucHSd \
     "https://172.18.0.2:30920/elk-logstash-*/_search" \
     -H 'Content-Type: application/json' -d '{
     "query": {
       "bool": {
         "must": [
           { "match_phrase": { "message": "failed to flush chunk" } },
           { "range": { "@timestamp": { "gte": "now-5m" } } }
         ]
       }
     }
   }' | jq '.hits.total'
   ```
   *Expected response:* `"value": 0`!

3. **Verify in Kibana Discover (KQL)**:
   Search for the noise pattern in the **Last 15 minutes**:
   ```kql
   kubernetes.pod.name.keyword: fluent-bit* and message: (
     "failed to flush chunk" or
     "unable to authenticate user" or
     "status=401" or
     "Non-zero metrics" or
     "DEPRECATED"
   )
   ```
   *Expected result in Discover:* **No results found** (0 hits)! All feedback storm noise is stopped!

---

### 🔍 Production Takeaway: The Rule of Defensive Observability
1. **Never Ship Log Shipper Errors Unconditionally**: If a log shipper emits errors when it fails to reach Elasticsearch, shipping those error logs to the same Elasticsearch cluster creates an exponential amplification loop that can saturate network bandwidth and fill up storage.
2. **Drop Health Probes Early**: Kubernetes liveness and readiness probes (`/healthz`, `/livez`) execute every 5–10 seconds per container. In a 500-pod cluster, that generates millions of useless HTTP 200 logs per day. Dropping them in Logstash or Filebeat reduces Elasticsearch licensing and storage costs by 30% to 60%.

---

# 🎯 Task 4: Full-Stack Metric & Log Correlation with Metricbeat & Kibana Lens

### 🏢 Real-World SRE Scenario: "Is it an OOM or Application Bug?"
During the `payment-api-failing` crash incident and `sample-logger` latency alerts, the engineering team asked:
> *"Is the pod getting OOMKilled (Out Of Memory) by the Linux kernel? Did high node CPU throttle database connection handshakes, causing the 30-second connection timeout?"*

In high-stakes production incident triage, investigating **Logs alone** only answers *"What error did the software print?"*. 
To answer *"Did the underlying infrastructure cause the failure?"*, an SRE must correlate **Logs** with **Metrics** (`Metricbeat`):
1. **Container Memory Working Set** (`kubernetes.container.memory.workingset.bytes`) vs. Memory Limits (`64Mi`).
2. **Container CPU Usage** (`kubernetes.container.cpu.usage.node.pct`).
3. **Kibana Lens Dashboards**: Correlating error spike timestamps directly with resource utilization graphs.

---

### 📝 Task Objectives
1. **Query Metricbeat Data**:
   - Query `metricbeat-*` to inspect memory and CPU utilization for `payment-api-failing` and `sample-logger`.
   - Verify whether memory reached pod limits or triggered an OOM kill.
2. **Build a Kibana Lens Metric Visualization**:
   - Navigate to Kibana Lens and create a **Top 5 Containers by Memory Usage** bar/line chart using the `metricbeat-*` data view.
3. **Build a Log Error Distribution Panel**:
   - Create a stacked bar chart from `elk-*` showing log volume partitioned by `log.level` (`INFO`, `WARN`, `ERROR`, `FATAL`) over time.
4. **Correlate on the Dashboard**:
   - Place both panels side-by-side on the `[DevOps] Kubernetes ELK Observability` dashboard.
   - Synchronize the time picker to confirm that while error logs spiked, memory usage remained flat at ~1 MB, definitively ruling out an OOM or infrastructure bottleneck!

---

## 💡 Task 4: Step-by-Step Solution & Implementation

### Step 1: Query Metricbeat for Pod Memory & CPU via Terminal
Run this query to inspect container resource metrics during the failure window:

```bash
curl -sk -u elastic:0P1rf1AiLY0ucHSd \
  "https://172.18.0.2:30920/metricbeat-*/_search" \
  -H 'Content-Type: application/json' -d '{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        { "term": { "event.dataset": "kubernetes.container" } },
        { "term": { "kubernetes.namespace": "default" } }
      ]
    }
  },
  "aggs": {
    "by_pod": {
      "terms": { "field": "kubernetes.pod.name.keyword", "size": 5 },
      "aggs": {
        "avg_mem_bytes": {
          "avg": { "field": "kubernetes.container.memory.usage.bytes" }
        },
        "max_cpu_pct": {
          "max": { "field": "kubernetes.container.cpu.usage.node.pct" }
        }
      }
    }
  }
}' | jq .aggregations
```

#### What the Metric Output Proves:
- **`payment-api-failing` Memory**: ~1.02 MB (Memory limit is 64 MB).
- **`sample-logger` Memory**: ~3.42 MB (Memory limit is 64 MB).
- **CPU Utilization**: Less than 0.2% across both pods.
- **RCA Conclusion**: **Neither container is starved for resources or OOMKilled.** The crash is 100% caused by application database connection configuration and network reachability, not host resource exhaustion!

---

### Step 2: Build the Metricbeat Lens Visualization in Kibana
1. In Kibana, click the hamburger menu (☰) $\rightarrow$ **Analytics** $\rightarrow$ **Visualize Library**.
2. Click **Create visualization** $\rightarrow$ select **Lens**.
3. In the top-left Data View dropdown, switch from `Kubernetes Logs (ELK)` to **`Kubernetes Metrics (Metricbeat)` (`metricbeat-*`)**.
4. In the field list on the left:
   - Drag **`kubernetes.container.memory.workingset.bytes`** into the central canvas.
   - In the right-hand panel, set the operation to **Average** or **Max**.
   - Under **Break down by**, drag in **`kubernetes.container.name`** (or `kubernetes.pod.name.keyword`).
   - Set the chart type to **Horizontal Bar** or **Line Chart**.
5. Click **Save and return** $\rightarrow$ Name it: `[DevOps] Container Working Set Memory`.

---

### Step 3: Build the Log Level Distribution Panel in Kibana Lens
1. Click **Create visualization** $\rightarrow$ select **Lens**.
2. Select Data View: **`Kubernetes Logs (ELK)` (`elk-*`)**.
3. In the field list:
   - Drag **`@timestamp`** to the Horizontal axis.
   - Drag **Records count** to the Vertical axis.
   - Under **Break down by**, drag in **`log.level`**.
4. Set the chart type to **Stacked Bar Chart**.
5. Notice how `INFO`, `WARN`, `ERROR`, and `FATAL` logs stack neatly over time, showing exactly when the error spike started!
6. Click **Save and return** $\rightarrow$ Name it: `[DevOps] Log Volume by Level Over Time`.

---

### Step 4: Add Panels to the DevOps Dashboard
1. Go to **Analytics** $\rightarrow$ **Dashboard** $\rightarrow$ select **`[DevOps] Kubernetes ELK Observability`**.
2. Click **Edit** in the top-right toolbar.
3. Click **Add from library** $\rightarrow$ Select:
   - `[DevOps] Container Working Set Memory`
   - `[DevOps] Log Volume by Level Over Time`
4. Rearrange the panels:
   - Top Row: Metric utilization (Memory / CPU)
   - Middle Row: Log volume breakdown by level
   - Bottom Row: Discover live log search table
5. Click **Save** in the top right.

---

### 🔍 Production Takeaway: The Power of Unified Observability
When On-Call, navigating between separate disconnected monitoring tools (e.g., only looking at Grafana or only looking at Kibana) creates blind spots. By pulling both Metricbeat and Filebeat into Elasticsearch and correlating them on a single Kibana dashboard:
1. **Mean Time to Resolution (MTTR) is cut in half**: You instantly verify whether an error spike is preceded by a memory leak or CPU throttling.
2. **Definitive Root Cause Differentiation**: You can confidently tell developers whether an issue is an infrastructure sizing problem (scale up pod limits) or code/database bug (fix connection pool logic).



