Introduction

In this tutorial, you'll discover how Elastic Observability provides comprehensive system visibility through the APM Service Map and integrated observability tools. Unlike the manual investigation required with kubectl, you'll see how logs, metrics, and traces are automatically correlated to answer operational questions in seconds.

Prerequisites: Completion of the previous tutorial (Investigating Without Observability)

**Key Terms**

<img width="1516" height="608" alt="image" src="https://github.com/user-attachments/assets/f1de20c6-232c-4eab-a09c-c709cea4d5a1" />

**Comprehensive Observability with Elastic**

1. Open the Service Map

Your Kibana tab should already be open and you should see the APM Service Map. Zoom in to see the nodes more clearly.

<img width="1049" height="669" alt="image" src="https://github.com/user-attachments/assets/f2c7b10f-2c31-49d9-a20b-34780111b43f" />

Take a moment to appreciate what you're seeing:

All services visualized - Every service you found with kubectl is shown as a node

Automatic discovery - You didn't configure anything; this was generated automatically from OpenTelemetry trace data

Service relationships - Lines show which services communicate with each other

Real-time updates - The map reflects actual system behavior

You can instantly see all services and their relationships.

🔍 Why this matters

In the previous challenge, you had to guess which services depend on each other by reading logs or examining code. The Service Map answers the question "which services does checkout depend on?" instantly—something that would take significant investigation with traditional tools.

2. Examine a service node

Click on the checkout service node. You'll see a summary panel showing:

Latency - Average response time

Throughput - Requests per minute

Failed transaction rate - Error percentage

<img width="466" height="467" alt="image" src="https://github.com/user-attachments/assets/98c99e3e-f3f4-4596-a3e7-f2be82efe5d5" />

The OpenTelemetry instrumentation automatically captures and aggregates comprehensive performance data that would be impossible to obtain through manual log analysis.

Why this matters

These three metrics—latency, throughput, and failed transaction rate—are the core indicators of service health. With kubectl, answering "is the checkout service healthy?" required manually counting log entries. Here, you get real-time aggregate metrics at a glance.

3. Focus on service dependencies

With the checkout service still selected, click Focus map.

<img width="798" height="412" alt="image" src="https://github.com/user-attachments/assets/38ab5cdd-3836-456b-89e8-7c83a652d6e9" />

The map highlights the checkout service and all services it directly communicates with:

cart (to retrieve cart contents)

shipping (to calculate shipping costs)

payment (to process payment)

email (to send confirmation)

currency (for price conversion)

4. View comprehensive service details

Click on the checkout service node again, then click Service details.

You'll see a comprehensive view with multiple visualizations:

Latency over time

Throughput over time

Transactions

Failed transaction rate

Errors

Dependencies

🔍 Why this matters

The time-series visualizations show trends, not just current values. You can see if latency is increasing, if error rates are spiking, or if throughput is declining—patterns that are nearly impossible to detect by reading individual log entries.

5. Explore service transactions

Scroll back up to Transactions and click on the oteldemo.CheckoutService/PlaceOrder transaction.

<img width="829" height="178" alt="image" src="https://github.com/user-attachments/assets/218d5120-91f1-47d9-9232-bcabd7030d29" />

This will take you to a comprehensive view of the transaction. This shows aggregate statistics for all PlaceOrder transactions:

<img width="1772" height="1396" alt="image" src="https://github.com/user-attachments/assets/1ed189e9-b197-4745-a980-3ba0812180e3" />

6. View a trace sample

Scroll down to the samples section and click View full trace.

<img width="1780" height="1390" alt="image" src="https://github.com/user-attachments/assets/38b42db1-8081-4cd6-8779-d826a3d723b5" />

You'll see a timeline visualization showing:

Every service involved in this specific request

Timing for each service (horizontal bars = time spent)

The call sequence (which service called which)

Total request duration

Scroll down to see the complete visualization. Note that measurements are collected down to a micro-second granularity.

You can see exactly which services were involved and where time was spent which would have been extremely difficult using only individual service logs.

🔍 Why this matters

This trace view answers questions that were impossible with kubectl: "Which services did this request touch?" and "Where did the time go?" You can see the exact sequence of calls and identify bottlenecks by looking at which spans took the longest.

7. Examine correlated logs

While viewing the trace, click the Logs tab.

<img width="818" height="630" alt="image" src="https://github.com/user-attachments/assets/b02961d4-1efb-47ad-8419-eefdf2db36e5" />

You'll see all logs from all services involved in this specific transaction, automatically correlated by trace_id.

Instead of manually collecting logs from multiple pods and trying to match timestamps, you can see all related logs in one view.

🔍 Why this matters

This is the correlation that was missing when you used kubectl. OpenTelemetry propagates a unique trace_id across all services involved in a request. Elastic uses this ID to automatically group all related logs, eliminating the manual timestamp-matching work you would otherwise need to do.

8. Search and filter logs

You can also use Elastic's powerful query language ES|QL to query the observability data stored in Elasticsearch.

Under Observability click on Discover and, once there, click on Try ES|QL.

<img width="1242" height="419" alt="image" src="https://github.com/user-attachments/assets/05365ea9-ec51-494f-9a49-11bf81d024d2" />

Use the following ES|QL query to retrieve the logs shipped by OpenTelemetry to Elasticsearch and stored in the logs-generic.otel-default index.

```bash
from logs-generic.otel-default
```

You will see logs from all services that have generated logs within the specified time window:

<img width="1259" height="718" alt="image" src="https://github.com/user-attachments/assets/ba77fbca-2d1a-4410-abb3-ed0751318529" />

Try filtering to just the logs of the checkout service:

```bash
from logs-generic.otel-default | where service.name == "checkout"
```

<img width="1286" height="726" alt="image" src="https://github.com/user-attachments/assets/5dac3dd3-b215-4ff7-9e89-4c22fae43428" />

You can now search across all service logs from one interface.

The other signals: metrics and traces, are also stored in Elasticsearch indices and can be queried using ES|QL in similar ways.

The three observability signals

You've now seen all three observability signals working together in Elastic:

Logs - Discrete events showing what happened

You explored logs in Discover and saw them correlated to traces

Logs provide detailed context about individual operations

Metrics - Numerical measurements showing how much

You saw latency, throughput, and error rates in the Service Map and service details

Metrics show trends and aggregate performance over time

Traces - Request flows showing where time went

You examined the trace timeline to see timing across services

Traces reveal the complete path of individual requests

Together, these three signals provide comprehensive visibility that kubectl alone cannot offer.

