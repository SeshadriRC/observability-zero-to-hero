**Introduction**

In this tutorial, you'll apply your observability skills to troubleshoot a realistic production problem. Using a systematic approach—Service Map to metrics to traces to span details—you'll identify why customers can't complete their orders. This hands-on investigation demonstrates how modern observability transforms hours of manual troubleshooting into minutes of systematic analysis.


<img width="1715" height="412" alt="image" src="https://github.com/user-attachments/assets/d14f18bb-b2c7-4772-8766-5602c3797342" />

**Problem Scenario**

The Astronomy Shop team has received customer complaints that customers are not able to place their orders. Customers are very unhappy and the business is losing revenue. Your job as the site reliability engineer (SRE) is to investigate and identify the root cause.

Background: The payment service was scaled down last night during a scheduled maintenance window. It should have been scaled back up before business hours, but that step appears to have been missed.

**Replicate the Problem**

1. Attempt to replicate the problem in the Astronomy Shop

The first thing you do as an SRE is use the information you have and attempt to replicate the problem. So:

- Go to the Astronomy Shop tab
- Select and add a product to the cart
- Attempt to place the order

You will notice that, no matter how many times you click Place Order, there is absolutely no response. The symptoms are clear: you can reach the checkout step, but cannot successfully complete it.

<img width="2098" height="980" alt="image" src="https://github.com/user-attachments/assets/927280fc-aeb9-4eeb-892f-8419466c0638" />

🔍 Why this matters

Replicating the reported problem is the first step in any troubleshooting workflow. Confirming the symptoms helps you understand what users are experiencing and gives you a baseline for verifying that your fix works.

**Troubleshoot**

1. Start troubleshooting with the Service Map

Switch to the Kibana tab which should already be in the Service Map.

<img width="1049" height="669" alt="image" src="https://github.com/user-attachments/assets/b514a101-abdf-4d84-8b78-a99a6df27662" />

🔍 Why this matters

The Service Map is your starting point for any investigation. It provides a system-wide view that immediately shows which services are experiencing problems, rather than requiring you to check each service individually.

2. Examine the checkout service

Click on the checkout service node to see its summary metrics and trends.

<img width="1736" height="1378" alt="image" src="https://github.com/user-attachments/assets/0e585270-3042-451c-b2da-aee9ccd0d87e" />

Wait a few minutes. You should see an elevated failed transaction rate, in this case 66%, and from the trend graph we can see that it's not intermittent and likely climbing. Most checkout transactions are indeed failing. We can also see an increase in latency and a decrease in throughput when compared to the baseline.

Recall that when we first explored the Service Map, the checkout service had no failed transactions.

3. Analyze the failed transaction rate

Click Service Details to open the comprehensive service view for the checkout service. Scroll down to the Failed transaction rate chart and you will see when the issue started (extend the time window to see when the failed transaction rate spiked).

<img width="1654" height="538" alt="image" src="https://github.com/user-attachments/assets/f4ad3831-cdd8-432b-9a03-c850257ab442" />

Clearly a significant event took place that is causing an outage in the checkout flow.

🔍 Why this matters

Time-series metrics help you establish when a problem started. Knowing the exact time of the spike can help correlate with change events (deployments, scaling operations, configuration changes) that might have caused the issue.

4. Explore checkout transactions

As you did previously, find the oteldemo.CheckoutService/PlaceOrder transaction.

<img width="1680" height="698" alt="image" src="https://github.com/user-attachments/assets/9d438aff-4c0d-4ac2-a000-7d5617d5f4ec" />

You can see here that the failed transaction rate is 100%.

5. View a failed transaction trace

Click to open the transaction.

<img width="410" height="225" alt="image" src="https://github.com/user-attachments/assets/c081df4c-289e-44f4-b8f4-c70807ab24f1" />

Now scroll down to the timeline diagram. This diagram is showing you a sample trace. In our case, it is showing a trace that ended in failure:

<img width="1694" height="1334" alt="image" src="https://github.com/user-attachments/assets/c636dfe6-bfff-4e0d-988f-4bfb5bbfe1c2" />

You can navigate to other sample traces if the one you see does not show the failure. This should be easy since by now all transactions are failing!

If the button is active, click View full trace to see the end-to-end flow across the services involved in the transaction:

<img width="1664" height="1398" alt="image" src="https://github.com/user-attachments/assets/25ec3c13-0a0e-43f8-a662-e039e2eddfa8" />

Scroll all the way down to find the call at the root of the failure:

<img width="1688" height="328" alt="image" src="https://github.com/user-attachments/assets/a96ddc9a-f9d0-4f3e-a27c-904ec1fb2cc4" />

🔍 Why this matters

The trace timeline shows you exactly where in the request flow the failure occurs. By scrolling to the end of the trace, you can identify which service call failed—narrowing your investigation from "something is wrong with checkout" to "something is wrong with a specific downstream call."

**Identify the Root Cause**

We can see the root cause now. It seems to be a call to the payment service Charge operation.

Click on it to explore its details and find out more.

1. Analyze the failed span in Discover

From the Span Details view, click View span in Discover:

<img width="1788" height="1228" alt="image" src="https://github.com/user-attachments/assets/9ae4ca21-c446-4af2-8343-01390756ddb3" />

In Discover you will see the trace data sent by OpenTelemetry containing the span details:

<img width="2250" height="954" alt="image" src="https://github.com/user-attachments/assets/c29e723e-f64b-47c0-b7c6-b55d3f8fb24e" />

Expand the document to view all fields:

<img width="920" height="473" alt="image" src="https://github.com/user-attachments/assets/4e8771cb-82b4-4f9b-889d-cde14e586ef3" />

Now search for the message field:

<img width="1080" height="682" alt="image" src="https://github.com/user-attachments/assets/528b155a-fcd2-4cb8-b8a0-82004e602bab" />

We have found the root cause! It appears to be a connection error.

```
"connection error: desc = ""transport: Error while dialing: dial tcp 10.43.128.174:8080: connect: connection refused"""
```

🔍 Why this matters

The "connection refused" error tells you that the checkout service attempted to connect to the payment service, but no process was listening on that port. This typically means the target service is not running—which aligns with the background information about the payment service being scaled down during maintenance.

2. Confirm the root cause

Based on your investigation, you've discovered:

From the Service Map: The checkout service shows high failure rates

From metrics: Failed transaction rate spiked to nearly 100% at a specific time

From traces: Errors occur when checkout attempts to call the payment service

From span details: Connection refused errors indicate the payment service is unreachable

Root cause: The payment service pod was likely scaled down during last night's maintenance window and was not scaled back up. The service is completely unavailable, causing all payment processing attempts to fail with connection errors.

<img width="1725" height="675" alt="image" src="https://github.com/user-attachments/assets/1d634238-b719-4b99-82a3-e3daea582fcc" />

<img width="1747" height="402" alt="image" src="https://github.com/user-attachments/assets/d80f0603-990d-44ed-9494-029bae62c9ab" />

<img width="1728" height="396" alt="image" src="https://github.com/user-attachments/assets/b01ad516-f72d-4ad8-b7da-855a206bd2ca" />

2. You can use the Service Map to observe how the failed transaction rate goes down as customers are able to place orders again.

<img width="499" height="444" alt="image" src="https://github.com/user-attachments/assets/629dc2b2-3256-44cb-ae0a-28f1c8bebf1f" />

If you wait long enough, it should go back down to 0% or the value you observed in your own baseline.

You can also (and should) go to the store and attempt to complete an order:

<img width="1100" height="770" alt="image" src="https://github.com/user-attachments/assets/9e896146-9908-44f8-84aa-178b3cf86be2" />

Success! Customers can place orders again. You found the root cause and resolved the issue using Elastic Observability.

Investigation time: With traditional kubectl-only troubleshooting, finding this issue could have taken 30-60 minutes of checking each service individually. With Elastic Observability, you identified the root cause in under 5 minutes by following a systematic approach through the Service Map, metrics, traces, and span details.

💡 Best practice

After resolving an incident, document what happened: the symptoms, root cause, resolution, and how long each phase took. This post-incident review helps improve processes and creates a knowledge base for future incidents. Many teams also add alerting rules to catch similar issues proactively—for example, alerting when the payment service has zero replicas.

In this tutorial, you investigated a real checkout failure affecting the Astronomy Shop using a systematic four-step approach: detect, scope, isolate, and diagnose.

First, you used the APM Service Map to identify that the checkout service had an elevated failed transaction rate.

Next, you examined the failed transaction rate chart to establish when the problem started, then followed a failing trace through the waterfall to isolate the failure to the payment service.

Finally, you examined the span details in Discover to find the connection refused error that confirmed the payment service was unavailable — and used kubectl to restore it.
