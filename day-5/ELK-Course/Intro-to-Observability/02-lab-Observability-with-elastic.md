Introduction

Objectives

In this lab, you will:

Navigate the APM Service Map to visualize service architecture

Identify key observability metrics for the checkout service

Explore service dependencies and transactions

Understand how distributed tracing enables log correlation

Setup

You should already be in the Kibana tab viewing the Service Map

The Service Map displays all services with trace data from the selected time window

Use the time range selector (top right) if you need to adjust the time window

**Task -1 Service Map Metrics**

<img width="1917" height="773" alt="image" src="https://github.com/user-attachments/assets/0a4cad3a-4b20-470a-8394-5fdec05069a3" />

Click on the checkout service node in the Service Map. Identify the THREE metrics shown in the summary panel.

Format: Submit the metrics exactly as they appear, separated by commas, including the "(avg.)" notation.

Example: Metric1 (avg.),Metric2 (avg.),Metric3 (avg.)

Hints

The three metrics appear in order from top to bottom in the summary panel
Each metric includes "(avg.)" in its name
Answer

```
Latency (avg.), Throughput (avg.), Fail
```

**Task -2 Checkout Transaction**


From the checkout service, navigate to Service Details and find the transaction name.

What to do:

With the checkout service selected, click Service Details

Scroll down to the Transactions section (or click the Transactions tab)

Find the checkout transaction name

<img width="1912" height="813" alt="image" src="https://github.com/user-attachments/assets/4f826bf8-6e3c-4d4d-ae7d-24c4997b29e4" />

**Task - 3  Service Dependencies**

Click "Focus map" on the checkout service and count how many services it directly depends on.

What to do:

Click on the checkout service node

Click Focus map button

Count the services that have direct connections (lines) to checkout

Hints

The Focus map highlights only services directly connected to checkout

Count the services that checkout calls to complete an order

Think about what's needed for checkout: cart contents, shipping calculation, payment processing, email confirmation, currency conversion, product information

<img width="1302" height="757" alt="image" src="https://github.com/user-attachments/assets/cd37481f-78e3-4485-845a-960952156616" />

```
6
```

**Task - 4  Opentelemetry Log Index**

In the tutorial, you used ES|QL to query logs. What is the Elasticsearch index name where OpenTelemetry logs are stored?

What to do:

Navigate to Observability → Discover

Click Try ES|QL

Recall the ES|QL query from the tutorial that retrieved all logs

```bash
from logs-generic.otel-default
```

The index name appears after the from keyword in ES|QL queries

OpenTelemetry logs use a naming pattern: logs-generic.otel-[namespace]

The namespace in this environment is default

<img width="1917" height="850" alt="image" src="https://github.com/user-attachments/assets/f36748de-8306-4670-8366-10910e957634" />

---

**Task - 5  Trace Correlation**

What field name is used to correlate logs across different services to the same trace?

What to do:

1. Think about what you learned in Challenge 1 - the "observability gap" was the inability to correlate logs

2. Distributed tracing solves this by adding a unique identifier to every log entry in a transaction

3. When viewing a trace's logs, this field connects them all together


** Hints **

This field uniquely identifies which trace a log entry belongs to

The field name contains two words separated by an underscore

In the tutorial, when you clicked the "Logs" tab in a trace view, logs were automatically correlated by this field

`Answer: Traceid`

Outstanding work! You've successfully completed this lab and demonstrated your ability to:

**Conclusion**

Navigate the APM Service Map to visualize service architecture.

Identify key observability metrics for the checkout service.

Explore service dependencies and transactions.

Understand how distributed tracing enables log correlation.

Next, you'll apply these skills to troubleshoot a real problem and find its root cause using the Service Map, metrics, traces, and correlated logs.

---
