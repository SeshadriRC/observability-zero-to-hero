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
| **Task 1** | **Triage & Querying** | Incident RCA: Diagnosing Crashing Services using KQL, ES\|QL & APIs | 🟢 Completed / Ready |
| **Task 2** | **Log Pipeline & Parsing** | Logstash Grok & ECS Extraction for Unstructured Logs (`payment-api`) | ⏳ Next |
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
