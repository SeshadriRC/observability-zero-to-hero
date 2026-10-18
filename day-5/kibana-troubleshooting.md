# 🛠️ Kibana Troubleshooting: "Kibana server is not ready yet"

## 🔍 1. Symptoms
- Accessing Kibana in the browser or via curl returns:
  ```http
  HTTP/1.1 503 Service Unavailable
  Kibana server is not ready yet
  ```
- Kibana pod logs (`kubectl logs -n logging -l app=kibana`) display errors like:
  ```text
  [ERROR][elasticsearch-service] Unable to retrieve version information from Elasticsearch nodes. security_exception: [index_not_found_exception] Reason: no such index [.security]
  ```
  or:
  ```text
  [ERROR][elasticsearch-service] Unable to retrieve version information from Elasticsearch nodes. 401 Unauthorized
  ```

---

## 🧠 2. Root Cause
In **Elasticsearch 8.x**:
1. Kibana authenticates to Elasticsearch using a **Service Account Token** stored in the Kubernetes secret `kibana-kibana-es-token`.
2. These service account tokens are stored internally inside Elasticsearch's **`.security`** index.
3. In local environments (like a Kind cluster where persistence is disabled or PVC is pending), whenever `elasticsearch-master-0` restarts, its ephemeral storage is reset, wiping the `.security` index.
4. Because the token record in `.security` is lost, Elasticsearch rejects Kibana's existing token with `401 Unauthorized`, leaving Kibana in a permanent waiting loop.

---

## ⚡ 3. Quick Action Plan (Run These 3 Steps)

### Step 1: Generate a fresh service token inside Elasticsearch
Run this command to create a new token for `elastic/kibana` and store it in a variable:
```bash
TOKEN=$(kubectl exec -n logging elasticsearch-master-0 -- \
  /usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana-token | awk '{print $4}')

# Verify the token is captured
echo "New Token: $TOKEN"
```
*(This command automatically re-creates and initializes the `.security` index if missing).*

### Step 2: Update the Kubernetes secret with the new token
Patch the `kibana-kibana-es-token` secret in the `logging` namespace:
```bash
kubectl create secret generic kibana-kibana-es-token -n logging \
  --from-literal=token="$TOKEN" \
  --dry-run=client -o yaml | kubectl apply -f -
```

### Step 3: Restart Kibana to apply the new secret
```bash
kubectl rollout restart deployment kibana-kibana -n logging
```

---

## ✅ 4. Verification

1. **Watch the rollout**:
   ```bash
   kubectl rollout status deployment kibana-kibana -n logging
   ```

2. **Verify pod is `1/1 Ready`**:
   ```bash
   kubectl get pods -n logging -l app=kibana
   ```

3. **Check Kibana API status**:
   ```bash
   ES_PWD=$(kubectl get secret -n logging elasticsearch-master-credentials -o jsonpath="{.data.password}" | base64 --decode)
   KIBANA_POD=$(kubectl get pod -n logging -l app=kibana -o jsonpath='{.items[0].metadata.name}')

   kubectl exec -n logging "$KIBANA_POD" -- \
     curl -sk -u "elastic:$ES_PWD" http://localhost:5601/api/status | jq .status.overall
   ```
   *Expected response:*
   ```json
   {
     "level": "available",
     "summary": "All services are available"
   }
   ```

4. **Port forward and access**:
   ```bash
   kubectl port-forward svc/kibana-kibana 5601:5601 --address 0.0.0.0 -n logging
   ```
