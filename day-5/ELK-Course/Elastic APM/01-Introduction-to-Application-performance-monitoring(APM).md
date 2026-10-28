Application Performance Monitoring (APM) with Elastic

Application Performance Monitoring (APM) provides the observability signals needed to understand and optimize your application's behavior. APM provides deep visibility into your application's behavior by tracing requests as they flow through your distributed services. While logs tell you what happened, metrics show you how your system is performing, and synthetic monitoring tells you whether users can accomplish their goals, APM reveals exactly how your application processes each user request and where performance bottlenecks or failures occur.

APM bridges the gap between high-level system health and detailed application behavior. When a user reports slow checkout times, for instance, APM traces show you exactly which services were involved, how long each operation took, and where the delay occurred—whether it was a slow database query, an external API call, or inefficient code execution. This request-centric view complements your other observability signals to provide complete visibility into user experience.

Elastic APM collects five primary types of data to paint a complete picture of application performance:

- Spans A span contains information about the execution of a specific code path. They are the building blocks of transactions and traces.
- Transactions A transaction describes an event captured by an Elastic APM agent instrumenting a service. A transaction is technically a type of span that has additional attributes associated with it and often contains multiple child spans. You can think of transactions as the highest level of work you’re measuring within a service. Transactions also identify service entry points.
- Traces A trace is a group of transactions and spans with a common root. Each trace tracks the entirety of a single request. When a trace travels through multiple services, it is known as a distributed trace. When a trace travels through multiple services, it reveals the interaction patterns and dependencies that drive overall system performance.
- Errors capture information about an exception that occurred or about a log created when the exception occurred.
- Metrics measure the state of a system by gathering information on a regular interval. There are two types of APM metrics: System metrics: basic infrastructure and application metrics. Calculated metrics: Aggregated trace event metrics used in the APM visualizations.

All APM data integrates seamlessly with your existing Elastic Stack, allowing you to correlate application traces with logs, infrastructure metrics, and synthetic monitoring results for complete observability.

In this track, you will:

- Identify how performance issues propagate through service dependencies and impact user experience
- Understand the APM data types and how they work together to provide application visibility
- Navigate the APM UI to explore transactions, spans, errors, and metrics in real applications
- Use the service map to visualize application architecture and identify service dependencies
- Simulate application failures and trace their impact through distributed services
- Leverage the AI Assistant for Observability to analyze APM data and troubleshoot performance issues
  
⚠️ You are starting with an Elastic deployment that already has APM data. If you start over, your progress will be lost.

⏳ The setup will take at least 30 seconds - we are ensuring the OTel demo application is sending APM data to your cluster. If you see a loading bar at the bottom of this screen, it may take longer.
