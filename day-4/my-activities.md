- Test all the endpoints

<img width="1917" height="395" alt="image" src="https://github.com/user-attachments/assets/fdfb0026-7691-4003-a5c8-3342df88e652" />

- Check in prometheus whether custom metric is available or not. It will not be visible, because we need to do **service discovery**.

<img width="1917" height="611" alt="image" src="https://github.com/user-attachments/assets/b8de17fb-5a0a-482f-872b-04c830836283" />

- There can be multiple applications, how prometheus knows which application it should monitor. it will fetch the metrics from `/metrics` endpoint.
  
- Apply the service monitor manifest

<img width="1917" height="230" alt="image" src="https://github.com/user-attachments/assets/6ac65234-66cb-4e36-8a0c-4341d8a1b929" />

- Its visible now

<img width="1917" height="740" alt="image" src="https://github.com/user-attachments/assets/e649d886-9a07-4051-845c-a0f92d9f9a64" />

<img width="1917" height="885" alt="image" src="https://github.com/user-attachments/assets/462a0c85-e150-490b-a670-ad843db687db" />

- Configure the `alertmangerconfig.yml` file to trigger the alerts.

- `Email-id` is updated.

<img width="1917" height="795" alt="image" src="https://github.com/user-attachments/assets/d76bd0a7-422c-43fc-ad58-a2026983511c" />

- generate a email password and update in the file.

<img width="1917" height="216" alt="image" src="https://github.com/user-attachments/assets/c51a0ac8-d046-4e22-8790-f72eb6121693" />

<img width="1412" height="642" alt="image" src="https://github.com/user-attachments/assets/4c133747-de91-49b8-b305-4aee24bec3a3" />

- Convert it into `base64` format and update it in the `secret` file.

<img width="1263" height="671" alt="image" src="https://github.com/user-attachments/assets/b6369766-9e63-4999-9f81-40d044a889c2" />

<img width="1287" height="160" alt="image" src="https://github.com/user-attachments/assets/9c06051a-aa44-4dbf-91f4-b8cfe4bcfe62" />


<img width="1915" height="697" alt="image" src="https://github.com/user-attachments/assets/166d46ef-9fe6-4488-8d17-fd2d584678e8" />

- Now apply the `alert-manager` and `secret` again.

<img width="1492" height="228" alt="image" src="https://github.com/user-attachments/assets/9e7dee6b-3147-49b1-97e0-4de5808f9094" />

- Alert triggered automatically.

<img width="1598" height="106" alt="image" src="https://github.com/user-attachments/assets/6743fd51-5f1d-4444-8f52-7b4ba1c2ca2f" />

<img width="1907" height="437" alt="image" src="https://github.com/user-attachments/assets/c7ff82f7-ba33-49cf-8cea-76dc38e2527b" />

- Now crash the pod and check it.

<img width="1492" height="278" alt="image" src="https://github.com/user-attachments/assets/fca0be42-f751-4571-ae64-a73fa4a0084b" />

<img width="1916" height="841" alt="image" src="https://github.com/user-attachments/assets/f6e11937-a499-4340-82ea-2639ca1385b8" />
