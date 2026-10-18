# 📊 Kibana Log Investigation & Analysis Guide (Hands-on EFK / ELK)

This guide documents how to generate test logs, verify log ingestion into Elasticsearch, and navigate Kibana Discover to filter, search, and analyze logs like a DevOps engineer.

---

## 🚀 1. Deploying the Sample Logging Application

To investigate log flow, deploy a lightweight container that generates continuous structured logs with varying log levels (`INFO`, `WARN`, `ERROR`).

### Manifest: `sample-logger.yaml`
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

### Apply and Verify Pod:
```bash
# Apply deployment
kubectl apply -f sample-logger.yaml

# Check pod status
kubectl get pods -l app=sample-logger

# Check raw container stdout logs
kubectl logs -l app=sample-logger -f
```

---

## ⚡ 2. CLI Sanity Check: Verify Logs in Elasticsearch

Before jumping into the Kibana UI, you can verify from the terminal that documents are being actively indexed.

### Check Indices & Document Count:
```bash
ES_PWD=$(kubectl get secret -n logging elasticsearch-master-credentials -o jsonpath="{.data.password}" | base64 --decode)

kubectl exec -n logging elasticsearch-master-0 -- \
  curl -sk -u "elastic:$ES_PWD" https://localhost:9200/_cat/indices?v
```
*Look for:*
- `elk-logstash-YYYY.MM.DD` (Filebeat $\to$ Logstash pipeline)
- `logstash-YYYY.MM.DD` (Fluent Bit pipeline)

### Query Elasticsearch Directly for Sample App Logs:
```bash
kubectl exec -n logging elasticsearch-master-0 -- \
  curl -sk -u "elastic:$ES_PWD" "https://localhost:9200/_search?q=sample-logger&size=2&pretty"
```

---

## 🖥️ 3. Opening Kibana & Setting Up Data Views

### Access Kibana:
```bash
kubectl port-forward svc/kibana-kibana 5601:5601 --address 0.0.0.0 -n logging
```
- **URL**: `http://localhost:5601`
- **Username**: `elastic`
- **Password**:
  ```bash
  kubectl get secret -n logging elasticsearch-master-credentials -o jsonpath="{.data.password}" | base64 --decode; echo
  ```

### Create Data Views (Index Patterns):
1. In Kibana, open the menu (☰) $\to$ **Management** $\to$ **Stack Management** $\to$ **Data Views** (under Kibana).
2. Click **Create data view**:
   - **Name**: `ELK Logs`, **Index pattern**: `elk-logstash-*`, Timestamp field: `@timestamp`
   - **Name**: `EFK Logs`, **Index pattern**: `logstash-*`, Timestamp field: `@timestamp`

---

## 🔍 4. Investigating Logs in Discover

### Navigation:
1. Open Menu (☰) $\to$ **Analytics** $\to$ **Discover**.
2. Select your desired Data View from the top-left dropdown (e.g. `elk-logstash-*`).
3. Set the **Time Filter** (top-right):
   - Choose **Last 15 minutes** (or **Last 1 hour**).
   - Click the auto-refresh icon (clock with arrows) and select **5 seconds** for live streaming.

---

## 🎯 5. Searching with KQL (Kibana Query Language)

Enter these queries in the top search bar to filter logs effectively:

| Objective | KQL Query |
| :--- | :--- |
| **Filter by Deployment** | `kubernetes.deployment.name : "sample-logger"` |
| **Filter by Namespace** | `kubernetes.namespace : "default"` |
| **Filter by Container** | `kubernetes.container.name : "logger"` |
| **Find Only Errors** | `kubernetes.deployment.name : "sample-logger" and message : "ERROR"` |
| **Search High Latency** | `message : "High latency"` |
| **Search Specific Order ID** | `message : "1005"` |
| **Filter Multiple Pods** | `kubernetes.pod.name : ("sample-logger*" or "payment-api*")` |
| **Exclude Noisy Logs** | `not kubernetes.namespace : "kube-system"` |

---

## 💡 6. Production Pro-Tip: Clean Up Your Table Columns

By default, Discover shows the entire raw `_source` JSON document, which is difficult to read quickly. Customize your columns:

1. Look at the left sidebar under **Available fields**.
2. Hover over each of the following fields and click the **`+`** button (**Add field as column**):
   - `kubernetes.pod.name`
   - `stream` (`stdout` vs `stderr`)
   - `message`
3. Click on the column header in the table to sort ascending/descending.
4. Click on any log row to expand and inspect the full JSON document structure.

---

## 🔄 7. EFK vs. ELK Schema Differences

When comparing logs in both Data Views, notice the structure:

- **ELK (`elk-logstash-*` from Filebeat)**:
  Uses the **Elastic Common Schema (ECS)**:
  - `agent.type`: `filebeat`
  - `kubernetes.pod.name`: Pod name
  - `kubernetes.container.name`: Container name
  - `log.file.path`: Absolute path on the node (`/var/log/containers/...`)
  - `message`: Original stdout string

- **EFK (`logstash-*` from Fluent Bit)**:
  Uses Fluent Bit's native format:
  - `log`: The log text
  - `stream`: `stdout` / `stderr`
  - `kubernetes.pod_name`: Pod name
  - `kubernetes.container_name`: Container name
  - `kubernetes.namespace_name`: Namespace
