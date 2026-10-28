**Welcome to Elastic Application Performance Monitoring!**

In this course, you will explore the OTel demo ecommerce application through the lens of Elastic Application Performance Monitoring (APM) to understand how user requests flow through distributed services. The application is already instrumented and sending telemetry data to your Elastic cluster.

**1. Explore the APM Services overview**

Kibana should already be in **Observability > APM > Services**. This view shows all the services in your application and their key performance indicators.

<img width="2228" height="1566" alt="image" src="https://github.com/user-attachments/assets/8cc4c950-320e-431c-ae49-341f0a1b1cd6" />

Take a moment to examine the services list. You should see multiple services from the OTel demo application including `frontend`, `checkout`, `product-catalog`, and others. Notice how each service shows:

- **Latency** (average response time)
- **Throughput** (requests per minute)
- **Failed transaction rate** (percentage of failed requests)

**2. Explore the product catalog service**

Click on the `product-catalog` service to view its performance data so far. Traffic to the demo store is generated in the background, so there should already be some activity.

<img width="1840" height="1452" alt="image" src="https://github.com/user-attachments/assets/34f79520-d997-41d3-aea4-9d3daa61f87e" />

The service overview page shows several of the five APM data types working together:

**Metrics** are displayed in the charts at the top:

- **Latency** trends over time
- **Throughput** showing requests per minute
- **Failed transaction rate** indicating service health

**Transactions** are shown in the table below. You can see two transaction types:

- **oteldemo.ProductCatalogService/GetProduct** - individual product requests

- **oteldemo.ProductCatalogService/ListProducts** - product catalog browsing

Notice how each transaction shows its own latency, throughput, and failed transaction rate metrics.

**Before proceeding:** What do you think these two transactions represent in the context of an ecommerce application?

**Answer**

`GetProduct` represents users viewing individual product details, while `ListProducts` represents users browsing the product catalog or category pages. These are core shopping behaviors that directly impact user experience.

**3. Examine spans within a transaction**

Click on the `oteldemo.ProductCatalogService/GetProduct` transaction to explore its internal operations.

<img width="1812" height="1460" alt="image" src="https://github.com/user-attachments/assets/28ad033b-0518-45cf-acf1-0afb9e235cc5" />

This transaction detail view shows you the metrics for this specific operation over time. Notice the latency is measured in microseconds (μs), indicating this is a very fast operation. You can see patterns in the throughput and that there are no failed transactions.

Scroll down to see the Latency distribution histogram, which shows you how response times are distributed across all executions of this transaction. Look for the Trace sample section below the histogram.

<img width="1818" height="1410" alt="image" src="https://github.com/user-attachments/assets/ae1ada9c-e046-4971-a1af-b4a275657a9b" />

This trace sample shows something fascinating: this transaction is part of a distributed trace that spans multiple services! You can see:

**Services involved:** Both `frontend` and `product-catalog` services

**Complete trace timeline:** Shows how a user request flows from frontend → product-catalog (you may need to click View full trace to see the complete service-to-service flow)

**Spans breakdown:** Individual operations within the complete request:

- HTTP calls from the frontend
- API route execution
- gRPC call to the product catalog service
- The actual GetProduct processing

**Key insight:** This demonstrates how traces, transactions, and spans work together:

- The **trace** represents the complete user request across services

- **Transactions** represent the work done in each service

- **Spans** show the individual operations within each transaction

Even though the product-catalog service processes the request in just 106 μs, you can see the complete journey took longer due to network calls and frontend processing.

4. Explore the service map to understand application architecture

Click on the **Service Map** tab in the navigation to see how all services in your application connect.

<img width="1876" height="1460" alt="image" src="https://github.com/user-attachments/assets/1aeffb9f-8b4b-4951-94c8-92929ed85774" />

The service map provides a visual representation of your application's architecture derived from the trace data you've been exploring. You can see the complete OTel demo ecommerce application:

- Service nodes: Each circle represents a service - notice the `product-catalog` service is highlighted since that's where you navigated from
- Connection lines: Show how services communicate (you can see the frontend → product-catalog connection from your trace sample)
- Complete architecture: The full ecommerce flow from `load-generator` → `frontend-proxy` → `frontend` → various backend services
- Service types: Different icons indicate service types (databases like `redis`, message queues like `kafka/orders`, microservices like `checkout`, `shipping` etc.)

This map is automatically generated from your APM data - no configuration needed. You can see how complex distributed systems really are, with services like checkout connecting to email, cart, shipping, and fraud-detection.

**Key insight:** The service map shows the same frontend → product-catalog connection you saw in the trace sample, but now in the context of your entire application architecture. This visualization helps you understand dependencies and identify critical paths in your system.

5. Interact with the service map

Click on the `product-catalog` service node in the map to see a quick health summary.

<img width="680" height="570" alt="image" src="https://github.com/user-attachments/assets/d5892b62-8450-4ed6-a4d7-2b56664a93f3" />

This popup provides instant access to key service health metrics without leaving the map view:

- **Latency (avg.):** 171 μs - consistent with what you saw in the transaction details

- **Throughput (avg.):** 126.5 tpm - showing current request volume

- **Failed transaction rate (avg.):** 0% - indicating healthy service operation

Notice the **Service Details** button - clicking this takes you back to the detailed service overview you started with. This demonstrates how the different APM views are interconnected, allowing you to seamlessly navigate between high-level architecture visualization and detailed performance analysis.

The **Focus map** option filters the service map to show just this service and its immediate dependencies, useful for isolating specific parts of complex architectures.

Congratulations! You've now seen all APM data types working together:

- Spans contain information about specific code path execution.
- Transactions are the highest level of work performed by a service -- distributed traces travel through multiple services.
- Traces are groups of transactions and spans with a common root
- Errors are created when something goes wrong with your application and contain enough information to help you find its root cause.
- Metrics measure the state of a system by gathering information on regular intervals.

You have also seen how the Services Map visualizes your application architecture, showing how services connect to each other to perform the functions of the application.
