**Introduction**

In this module, you'll troubleshoot why APM traces aren't reaching your Elastic cluster.

First, in this lesson, you'll explore the OTel Demo microservices architecture, access the OTel Collector pod, and discover the secret token needed to authenticate with the APM Server. Understanding this data flow—from APM agents through the OTel Collector to Elastic—is fundamental to monitoring distributed applications.

**Key Terms**

<img width="1697" height="523" alt="image" src="https://github.com/user-attachments/assets/257ab216-5ebd-4947-9e16-fb44cf23a4a5" />

**OTel Demo Architecture and APM instrumentation (with OTel Collector)**

Below we explore the OTel Demo application and architecture, which is an application split into microservices.

The application is already instrumented with APM OpenTelemetry Agents. These agents are sending telemetry data to an OTel Collector agent. To get started, review the simplified overview architecture diagram below:

<img width="2324" height="1450" alt="image" src="https://github.com/user-attachments/assets/c38f3e33-12f8-4842-b69b-b0ff45942050" />

Notice there are currently no traces that are reaching your Elastic cluster under the **Observability Applications app.**

<img width="2738" height="2090" alt="image" src="https://github.com/user-attachments/assets/e260be6d-0dd5-4ca7-81d2-b9e400a0ca26" />

Warning

Please do not use the "Add data" button.

Let's focus on the communication between the OTel Collector and the APM server. From the Kibana main menu, navigate to Management > Integrations and select the APM integration. Scroll down and select the OpenTelemetry tab. You should see the following settings table:

<img width="2498" height="1480" alt="image" src="https://github.com/user-attachments/assets/c365eb14-5749-4230-a320-77e51f32c4b8" />

<img width="2108" height="790" alt="image" src="https://github.com/user-attachments/assets/a13de2fd-6f1f-4bdc-af9d-4572efa70ef8" />

🔍 Why this matters

The APM integration shows you the endpoint URL and authentication requirements that external collectors must use to send data. When traces aren't appearing, the first step is verifying that the collector's configuration matches these settings—particularly the authentication token.

Note

"When defined, secret tokens are used to authorize requests to the APM Server. Both the APM agent and APM Server must be configured with the same secret token for the request to be accepted." [Secret token | Elastic Docs]

The EDOT Collector supports Elastic API Key Authentication or Bearer Token Authentication. [Authentication methods for the EDOT Collector | Elastic Agent release notes]

Your Mission

- Discover which process runs OTel collector by accessing the pod and listing the processes.
  
- Find the value of the secret token used in the header authorization by accessing the configuration file of the running process.

🔍 Why this matters

Troubleshooting observability pipelines often requires inspecting the actual running configuration rather than relying on documentation. By accessing the collector pod and examining its configuration file, you can see exactly what credentials and endpoints it's using—skills that transfer to debugging any containerized observability component.

1. Navigate to the `Terminal` tab.

2. List the Kubernetes pods running in the `opentelemetry-operator-system` namespace. This command helps you identify which pods are running and which one you can connect to.

```bash
  kubectl get pods -n  opentelemetry-operator-system
```

💡Hints to access the OTel Collector

<img width="1614" height="278" alt="image" src="https://github.com/user-attachments/assets/ff0007cb-0d43-45e5-92d2-7c58da9b12a3" />


3. From the previous step, use the name of the pod named `opentelemetry-kube-stack-daemon-collector-<pod_id>` and access the pod with the following command:

```bash
kubectl exec -t -i <pod-name-id>  -n opentelemetry-operator-system  -- /bin/bash
```

💡Hint

This opens an interactive shell inside a running Kubernetes pod in the `opentelemetry-operator-system` namespace. From now on you're working inside the container.

<img width="2264" height="202" alt="image" src="https://github.com/user-attachments/assets/e4f2e118-076e-4bf9-ad07-b957fb953a9d" />

4. List all processes currently running inside the pod.

```bash
ps -ef
```

💡Hint to list the processes running

<img width="2144" height="304" alt="image" src="https://github.com/user-attachments/assets/a7f7cc87-b450-4389-8451-fb508894a11c" />

5. Filter the process list to show only OpenTelemetry-related processes.

```bash
ps -ef | grep -i otel
```

💡Hint to identify the process running

`ps` : report a snapshot of the current processes.

`ps -e`: Select all processes.

`ps -f`: Do full-format listing. man7.org/linux/man-pages

<img width="1636" height="168" alt="image" src="https://github.com/user-attachments/assets/92082895-dd95-4e23-9a2d-f8b6da4c6b78" />

Notice that the process running the OTel Collector is elastic-agent otel --config=/conf/collector.yaml

This confirms that:

- `elastic-agent` is running in OTel Collector mode.

- The collector configuration file is located at `/conf/collector.yaml`.

Although this is an OpenTelemetry Collector, it is actually implemented as an Elastic Agent process.

**Important**

"Elastic Agent includes an embedded EDOT Collector that allows you to instrument your applications and infrastructure once, and send data to multiple vendors and backends." Run [Elastic Agent as an EDOT Collector reference of the elastic-agent in OTel mode](https://www.elastic.co/docs/reference/fleet/otel-agent)


6. Inspect the collector configuration file to confirm that Bearer Token authentication is being used and retrieve the token value.

```bash
grep -i authorization  /conf/collector.yaml
```

💡Hints to access the OTel Collector configuration

<img width="1440" height="134" alt="image" src="https://github.com/user-attachments/assets/3f1b5a71-eb61-4f0a-a9ca-b57ea7f7e31b" />

From the output, you can identify the bearer token used for authentication: `nonprodpwd`

💡Hints to identify the value of the 'secret token'.

Search for a setting similar to this line:

```bash
Authorization: Bearer <token>
```

you can use `grep` command:

```bash
grep Authorization  <file>
```

Note

"Bearer authentication (also called token authentication) is an HTTP authentication scheme that involves security tokens called bearer tokens. The name "Bearer authentication" can be understood as "give access to the bearer of this token." "

[Bearer Authentication](https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/)

Conclusion

In this lesson, you traced the flow of APM data through the OTel Demo architecture, verified the OTel Collector configuration, and identified the authentication token required for traces to reach Elastic APM.

Click **Next** to continue.
