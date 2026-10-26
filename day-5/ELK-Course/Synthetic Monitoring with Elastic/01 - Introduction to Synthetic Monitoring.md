Welcome to Synthetic Monitoring with Elastic!

In this course, you will configure and deploy synthetics monitors to observe the OTel demo ecommerce application. Before we can do that, we need to have at least one location available. You can use Elatic's global infrastructure using Elastic Cloud. However, in this tutorial, you will be using a private location.


1. Create a private location for monitor deployment

Kibana should already be in Observability > Synthetics > Monitors. Click Create location to get started.

<img width="2248" height="1540" alt="image" src="https://github.com/user-attachments/assets/b2a42d26-c7ad-4a40-9fe4-93ca791ba084" />

Name the location `private_location` and use the `Elastic Agent on ECK policy`.

<img width="1074" height="1392" alt="image" src="https://github.com/user-attachments/assets/5c09b609-fe47-4a26-ac14-dda409e2ecc8" />

Click `Save` to finish creating the private location.

2. Create your first HTTP monitor

Now, let's create a lightweight HTTP monitor to ensure that the ecommerce store is online and responds to HTTP requests. From the left-side menu,

a. Click Monitors

<img width="444" height="248" alt="image" src="https://github.com/user-attachments/assets/9e4181a2-3ad6-4a84-873b-d30374faa8da" />

b. Click select a different monitor type.

<img width="1250" height="944" alt="image" src="https://github.com/user-attachments/assets/320fa623-e6f6-4214-89ea-4bb463f8b548" />

c. Select HTTP Ping as the monitor type.

<img width="1832" height="1010" alt="image" src="https://github.com/user-attachments/assets/85c96484-554e-4e9c-82c5-15c00a4ee953" />

d. Use `http://frontend-proxy-nodeport:8080` as the website URL.

<img width="1772" height="1420" alt="image" src="https://github.com/user-attachments/assets/0923c565-32b3-4e2e-8d94-3b7a996b4a49" />

e. Name the monitor `otel_ecommerce_store_http_monitor`.

f. Select the `private_location` you just created.

g. Leave the rest of the defaults. Note that the monitor will run every `3` minutes. You can change the frequency if you want to see results more quickly during this tutorial.

h. Click `Run test` to make sure the monitor executes without errors.

<img width="1886" height="312" alt="image" src="https://github.com/user-attachments/assets/85fb9cf2-955e-4d40-ad8b-ef71e65547cd" />

If everything is working fine you should get the following panel:  

<img width="1062" height="598" alt="image" src="https://github.com/user-attachments/assets/d51acfb7-c0f3-4ab5-a42a-9a7a1f858006" />

<img width="1912" height="843" alt="image" src="https://github.com/user-attachments/assets/9b271352-6ad2-4d99-86cb-4068cb3a36b3" />

<img width="811" height="277" alt="image" src="https://github.com/user-attachments/assets/105428d6-7ec1-4e38-9d8d-fef7830f1b21" />

Close the panel.

a. Click **Create monitor** to complete the process. You should see the monitor listed in the **Monitors** view.

<img width="1906" height="1244" alt="image" src="https://github.com/user-attachments/assets/85354021-f8f4-4d92-9b93-f63b27028645" />

3. Configure response body capture for diagnostics

By default, the monitor captures the body of the HTTP response only when there is an error. Let's change this:

a. Select the `otel_ecommerce_store` monitor and click `Edit monitor`.

<img width="754" height="902" alt="image" src="https://github.com/user-attachments/assets/af361edd-ab18-45bd-ae97-0b49adbd7d32" />

b. Under Advanced options, navigate to Response configuration and change Index response body to `Always`.

<img width="1870" height="612" alt="image" src="https://github.com/user-attachments/assets/3fe6e4df-e551-4bc8-920c-bd9c1d8a7936" />

c. Test the monitor and verify that the body is being captured.

<img width="1068" height="1370" alt="image" src="https://github.com/user-attachments/assets/9f9fad7e-6143-47d1-9880-cfd65bd943a4" />

d. Close the panel and click `Update monitor` to complete the process.

4. Test outage detection and recovery
   
Next, let's make sure the monitor will detect a store outage. Using the terminal, run this command to stop the `frontend` pod.

```bash
kubectl scale deployment frontend --replicas=0 -n default
kubectl scale deployment frontend-proxy --replicas=0 -n default
```

Back in Kibana, run a test manually if you don't want to wait the three minutes it takes the monitor to run. Note that it fails once it times out waiting for the store to respond.

<img width="1932" height="1394" alt="image" src="https://github.com/user-attachments/assets/f2d45e85-6b28-465e-ac04-00d742870781" />

Use the following command to restore the `frontend`:

```bash
kubectl scale deployment frontend --replicas=1 -n default
kubectl scale deployment frontend-proxy --replicas=1 -n default
```

Run a manual test to verify that the `frontend` is responding again.

View the History of the monitor to observe the store's behavior during the outage.

5. Create a monitor status alert

To truly be proactive, we need to create an alert that will notify us when the monitor fails a specific number of times within a time period. Follow these steps to create a monitor failure alert.

a. Click to expand Alerts and rules:

<img width="894" height="136" alt="image" src="https://github.com/user-attachments/assets/aebcce8e-afa7-4519-a53e-4ec0b05a0af3" />

b. Select Monitor status rule:

<img width="550" height="316" alt="image" src="https://github.com/user-attachments/assets/99365890-1402-430f-905e-a78a137e53de" />

c. Select Create status rule:

<img width="512" height="246" alt="image" src="https://github.com/user-attachments/assets/cb503f40-949f-4487-af0b-84212e822ccc" />

d. Leave the default name, but select the `otel_ecommerce_store_http_monitor`:

<img width="1090" height="1292" alt="image" src="https://github.com/user-attachments/assets/5487333a-9ab9-43f5-9901-9a80d8605aaa" />

e. Scroll down to add an Email action to the rule

<img width="1030" height="390" alt="image" src="https://github.com/user-attachments/assets/a865d3ab-f5cb-486f-accf-3a82bcd286f0" />

You will be using a preconfigured `Demo emails` connector:

<img width="1094" height="1070" alt="image" src="https://github.com/user-attachments/assets/9c4dc874-ebf8-43c1-8e10-10914cf36da1" />

f. Enter your email and a subject line and leave the rest of the defaults, but feel free to change body of the email:

<img width="1070" height="956" alt="image" src="https://github.com/user-attachments/assets/e3aa871b-457b-4cd0-aabd-f8072b075377" />

g. Save the alert and go to Stack Management > Rules where you should see the monitor status rule:

<img width="2242" height="1440" alt="image" src="https://github.com/user-attachments/assets/838c6a58-f082-44a0-a5f2-a5b33cfbe1a9" />

h. Select the monitor status rule:

<img width="1750" height="1428" alt="image" src="https://github.com/user-attachments/assets/976edf10-4745-4216-9fcc-92aa47df2646" />

i. Select the History tab where you should see that the eCommerce store has been responding to HTTP pings.

<img width="1738" height="1348" alt="image" src="https://github.com/user-attachments/assets/de45fd3c-42b1-4cfa-97a7-a51bb898093e" />

<img width="1912" height="827" alt="image" src="https://github.com/user-attachments/assets/2a693e70-5103-474a-b024-03b37dd83999" />


6. Trigger the alert

a. Use the same approach as before to stop the front end:

```bash
kubectl scale deployment frontend --replicas=0 -n default
kubectl scale deployment frontend-proxy --replicas=0 -n default
```

b. Now observe how the rule fails and how alerts are triggered:

<img width="1788" height="886" alt="image" src="https://github.com/user-attachments/assets/8bbe6e2a-b07b-471d-8caa-4319d98c248a" />

c. Finally, check your email to verify that the action worked properly (you may have to check your spam folder):

<img width="2588" height="770" alt="image" src="https://github.com/user-attachments/assets/1103af6e-b38c-4c5c-aa64-a6acf6900b12" />

Congratulations!

You've successfully created your first synthetic monitor and experienced how it detects real application outages. In this challenge, you:

- Set up a private location to run monitors from your own infrastructure

- Created an HTTP monitor that continuously checks your application's availability

- Configured response capture to gather diagnostic information

- Simulated a realistic outage and saw how synthetic monitoring provides immediate detection

- Explored monitor history to understand how failures are tracked over time

- Created and tested a monitor status alert to proactively be notified when the eCommerce store fails to respond to an HTTP ping.

Key Takeaway: Synthetic monitoring provides an outside-in perspective that complements your logs, metrics, and traces. It answers the critical question: "Can my users actually accomplish their goals?"

In the next lab, you'll expand your monitoring coverage by creating TCP and ICMP monitors to test different aspects of your application's health. These lightweight monitors work together to provide comprehensive coverage of your infrastructure.

Ready to continue? Click Next to practice what you've learned with additional monitor types.
