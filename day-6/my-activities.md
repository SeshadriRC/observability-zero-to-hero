## Travel example

<img width="1872" height="820" alt="image" src="https://github.com/user-attachments/assets/e30d776a-085b-4e05-b163-9d9277266dbc" />

## Developers and Devops team responsibilities

<img width="1162" height="453" alt="image" src="https://github.com/user-attachments/assets/f3d2e90e-839b-4078-b74d-1b696156bed1" />

## Comparison of Architecture

<img width="1032" height="496" alt="image" src="https://github.com/user-attachments/assets/f9feab55-cf41-494e-a7a8-99df06970b4f" />
<img width="810" height="312" alt="image" src="https://github.com/user-attachments/assets/ca02da99-5bb7-4d7f-ade6-73b252297720" />

---

```bash
To access the query UI:
  export POD_NAME=$(kubectl get pods --namespace tracing -l "app.kubernetes.io/instance=jaeger,app.kubernetes.io/component=all-in-one" -o jsonpath="{.items[0].metadata.name}")

  kubectl port-forward --namespace tracing $POD_NAME 16686:16686 --address 0.0.0.0 &

  Visit http://127.0.0.1:16686/
```

<img width="1917" height="426" alt="image" src="https://github.com/user-attachments/assets/893fe92c-ee93-43a2-bb2f-009275571705" />

- Before loading the application

<img width="1917" height="741" alt="image" src="https://github.com/user-attachments/assets/3beea101-963a-4390-aa09-6a1e3be66764" />

- After loading it you will see this traces

<img width="1917" height="852" alt="image" src="https://github.com/user-attachments/assets/97877977-9758-47ae-97f4-bad70c9436a4" />

- Now call service B and check it

<img width="1917" height="443" alt="image" src="https://github.com/user-attachments/assets/f47a2e16-3713-406f-a75f-db8fc00fd75c" />

<img width="1917" height="871" alt="image" src="https://github.com/user-attachments/assets/f5003c3b-856e-4eb0-bb17-e10af113d4d1" />
