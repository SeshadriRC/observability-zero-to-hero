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
