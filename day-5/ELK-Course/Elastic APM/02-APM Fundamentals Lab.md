APM Fundamentals Lab

In this lab you will practice using Elastic APM to investigate the OTel Demo eCommerce application performance and architecture.

02 - APM Fundamentals - Lab

**Lab Challenge: APM Service Analysis**

You've learned how to navigate APM to understand application performance using services, transactions, traces, and service maps. Now practice these skills by analyzing the OTel demo application to identify key performance characteristics.

**Your Mission**

Use APM navigation to identify the fastest and slowest services in the application, find a complex distributed trace, and map the critical service dependencies.

Goal: Complete specific analysis tasks that help you understand this application's performance profile using the APM navigation skills from the tutorial.

**Requirements**

Complete these analysis tasks:

✅ Identify the application service with the best performance (lowest latency among services handling requests)

✅ Identify the application service with the highest latency

✅ Find a distributed trace that spans at least 3 different services

✅ Use the service map to identify which service has the most connections

✅ Find the entry point for a checkout transaction by using "View full trace"

Hints to Guide You

Getting Started:

- Navigate to Observability > APM > Services in Kibana

- Use the latency column to compare service performance

- Look for services with interesting names like `frontend`, `checkout`, `cart`

**Task 1: Performance Leaders**

- Compare latency values in the Services overview for services that are actively handling requests
- Look for services with throughput data (not "N/A") - these are your application services
- Services with "0 ms" and "N/A" throughput are typically infrastructure services (like kafka, elasticsearch)
- Focus on application services that show actual request processing

**Task 2: Distributed Trace Hunting**

- Services that coordinate with others (like checkout or frontend) often have multi-service traces

- Navigate to a service's Transactions tab and examine transaction samples

- Look for traces that show multiple colored bars representing different services

**Task 3: Service Map Analysis**

- Use the Service Map tab to visualize the entire application
  
- Count the connection lines going into and out of each service
  
- Services in the center of the map often have the most connections

**Task 4: Entry Point Discovery**

- Navigate to the `checkout` service and find the PlaceOrder transaction

- Click on a transaction sample to view its trace details

- Use "View full trace" to see the complete request flow from the beginning

- Identify which service actually initiated this checkout request

**Navigation Reminders:**

- Click service names to view details

- Use the Transactions tab within services

- Click transaction samples to view traces

- Switch to Service Map tab for architecture view

**Success Criteria**

You've successfully completed the lab when you can:

- Name the fastest and slowest services with their approximate latencies

- Describe a distributed trace showing multiple services working together

- Identify the most connected service in the application architecture

- Follow a downstream transaction back to its entry point using traces

**Ready to analyze?** Start with the Services overview and work through each task systematically.

💡 Need help? Click here for example analysis results

Example Analysis Results:

**Task 1: Performance Analysis**

- **Infrastructure services:** kafka, coredns, elasticsearch (0ms with N/A throughput) - these support the platform but don't handle user requests

- **Fastest application service:** To find the service with the lowest latency, sort the services table in ascending order by latency. Ignore the infrastructure services. In this example, product-catalog with latency around 0.2ms and active throughput.

<img width="1380" height="1466" alt="image" src="https://github.com/user-attachments/assets/93bede95-6076-4497-a68d-5f39f6e1db03" />

  - Why fast: Simple database lookups for product information

- **Slowest application service:** Often `checkout` or `accounting` with latencies of 60-75ms. In this example, `fraud_detection` shows a latency of `10` minutes!

<img width="1396" height="1148" alt="image" src="https://github.com/user-attachments/assets/7572cd1c-9470-4ed8-871f-c4f0decfaa26" />

- Why slower: Complex business logic, payment processing, inventory checks

**Key insight:** When analyzing performance, focus on services with active throughput - these are handling real user requests.

**Task 2: Distributed Trace Example**

- Multi-service trace: Navigate to `checkout` service → Transactions → PlaceOrder

<img width="1336" height="1404" alt="image" src="https://github.com/user-attachments/assets/3eeb672a-94aa-4c8f-ba2b-ace23e45ac63" />

- **Transactions involved:** `checkout`, `cart`, `product-catalog`, `currency`, `shipping`, `quote`, `payment`, `email`, `fraud-detection`
  
- **What this shows:** A single user action like `checkout` requires coordination between multiple services

**Task 3: Service Connections**

- Most connected: Usually `frontend` or `checkout` services in this application (4-6 connections)

<img width="1358" height="930" alt="image" src="https://github.com/user-attachments/assets/a211e1b9-c800-4868-b080-b391c3d5c35f" />

- **Why:** Act as orchestrators connecting multiple services to achieve a goal

- **Connections to:** The `frontend` service connects to product-catalog, cart, checkout, recommendation, user services

**Task 4: Entry Point Discovery**

- **Starting point:** Navigate again to checkout service → PlaceOrder transaction sample

- **Trace exploration:** Click View full trace to see the complete request flow

<img width="1982" height="1472" alt="image" src="https://github.com/user-attachments/assets/b08d02db-8e09-405b-863d-aa39f589cf25" />

- Entry point identified: You can see that, in this example, the trace starts from load-generator service -- a reasonable origin since the traffic for the demo store is indeed simulated by a load generator

- Key insight: What looks like a checkout operation is actually part of a larger user journey that started elsewhere

**Key Learning:** Distributed traces help you understand the complete story of a request, not just individual service operations. The checkout PlaceOrder you see is the result of a user action that originated much earlier in the request flow.
