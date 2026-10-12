## Prometheus
 
 - Kube state metrics continuously looking the kube api server, so that it will have information of latest objects created. and it will store it in the /metrics endpoint

- Api server --> /metrics --> PromQL

```
to check the information node exporter is collecting
     clusterip of the node 
curl 10.96.88.17:9100/metrics   # /metrics is the endpoint

in the same node we can run curl command for kubestate metrics

curl 10.96.157.225:8080/metrics | grep container | restart
kube_pod_container_status_restarts_total
kube_pod_container_status_restarts_total{namespace="monitoring"}
kube_pod_container_status_restarts_total{namespace="default"} -> initially it won't show anything

Run a pod that is always crashing
kubectl run busybox-crash --image=busybox -- /bin/sh -c "exit 1"

kube_pod_container_status_restarts_total{namespace="default"} -> After crashing it will show

- also we can see how many configmap created in last 5 mins or how many secrets.
- cpu, memory usages
```

<img width="1917" height="707" alt="image" src="https://github.com/user-attachments/assets/dabe85ad-00a0-435c-acfc-9018eabc3a3f" />


---

## Grafana

- Better monitoring
- Authentication and Authorization


- In Adminstration tab, you can see authentication.

<img width="1856" height="722" alt="image" src="https://github.com/user-attachments/assets/c1d8bdc4-2dc8-4584-9472-958145022317" />

- Connections --> Datasources ( prometheus will get autoselected )

<img width="1917" height="717" alt="image" src="https://github.com/user-attachments/assets/cc85cd37-dd98-4ecc-bcd7-49c3314302df" />

- follow below steps to setup dashboard

```bash
New button --> new dashboard --> configure visualization --> click code in sql instead of builder --> paste below promql query and run it

kube_pod_container_status_restarts_total{namespace="default"}
```

<img width="1917" height="871" alt="image" src="https://github.com/user-attachments/assets/23c4ac54-db34-44bb-9d6c-cfd1b54cfd51" />

