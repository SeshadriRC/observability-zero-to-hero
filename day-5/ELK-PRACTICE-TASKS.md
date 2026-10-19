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

## 📋 Workshop Roadmap

| Task | Domain | Focus Area | Status |
| :--- | :--- | :--- | :--- |
| **Task 1** | **Triage & Querying** | Incident RCA: Diagnosing Crashing Services using KQL, ES\|QL & APIs | ✅ Completed |
| **Task 2** | **Log Pipeline & Parsing** | Logstash Grok & ECS Extraction for Unstructured Logs (Gin HTTP & Payment API) | 🟢 Active / Ready |
| **Task 3** | **Ingestion Optimization** | Storage Cost Control: Noise Drop Filters (Suppressing internal retry loops) | ⏳ Upcoming |
| **Task 4** | **Full-Stack Correlation** | Correlating Log Spikes with Node & Pod Resource Metrics (Metricbeat) | ⏳ Upcoming |
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

