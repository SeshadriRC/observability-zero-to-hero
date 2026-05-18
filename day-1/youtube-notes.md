# Observability Zero to Hero — Day 1 Summary

## Course Overview

This is a 7-day observability series focused on Kubernetes-based environments.

Topics covered across the series:

1. Fundamentals of Observability
2. Monitoring with Prometheus
3. PromQL (Prometheus Query Language)
4. Custom Metrics + AlertManager
5. Logging with EFK Stack
6. Distributed Tracing with OpenTelemetry & Jaeger
7. eBPF and modern observability

Notes, YAML files, scripts, and diagrams are available in the GitHub repository for each day.

---

# What is Observability?

Observability means understanding:

* The state of your application
* Infrastructure health
* Networking behavior

It helps answer:

* **What** is happening?
* **Why** is it happening?
* **How** to fix it?

Examples:

* CPU utilization
* Memory usage
* Disk utilization
* Failed HTTP requests
* Latency
* Request tracing

---

# Why Observability is Important

Without observability:

* You only know the system failed.
* You cannot identify the root cause quickly.

With observability:

* Detect issues early
* Find root cause faster
* Troubleshoot production problems
* Prevent SLA/SLO violations
* Improve application reliability

Used heavily in:

* Kubernetes environments
* Banking applications
* SaaS platforms
* High-traffic systems

---

# Three Pillars of Observability

## 1. Metrics

Metrics provide historical numerical data about the system.

Examples:

* CPU usage
* Memory usage
* Disk usage
* HTTP request count
* Error rate

Purpose:

* Understand system behavior over time
* Detect spikes/trends
* Generate alerts

Metrics answer:

> “What is happening?”

---

## 2. Logs

Logs provide detailed application events.

Types:

* Info logs
* Debug logs
* Error logs
* Trace logs

Purpose:

* Identify why failures occurred
* Understand application flow
* Debug issues

Logs answer:

> “Why is it happening?”

---

## 3. Traces

Tracing tracks request flow across services.

Example request path:

```text
Client → Load Balancer → Frontend → Backend → Database
```

Tracing shows:

* Which service handled the request
* Request latency between components
* Where failures occurred

Purpose:

* Root cause analysis
* Performance troubleshooting

Traces answer:

> “How do we fix it?”

---

# Metrics vs Logs vs Traces

| Pillar  | Purpose                              |
| ------- | ------------------------------------ |
| Metrics | Detect issues                        |
| Logs    | Understand failures                  |
| Traces  | Follow request flow and debug deeply |

Together they form complete observability.

---

# Monitoring vs Observability

## Monitoring

Monitoring mainly focuses on:

* Metrics
* Alerts
* Dashboards

Typical monitoring stack:

* Prometheus
* Grafana
* AlertManager

Monitoring is a subset of observability.

---

## Observability

Observability includes:

* Metrics
* Logs
* Traces

It provides deeper debugging and root cause analysis.

---

# Real-Time Use Case

Example:
A company hosts a Resume Builder application on Kubernetes in AWS.

They promise customers:

* 99.9% uptime
* Fast response times
* Minimal request failures

To maintain these SLAs/SLOs:

* Metrics detect failures
* Logs identify errors
* Traces locate bottlenecks

Observability helps prevent production outages before customers notice them.

---

# Role of Developers vs DevOps Engineers

Observability is a shared responsibility.

## Developers

Responsible for:

* Instrumenting metrics
* Writing logs
* Implementing traces

Using:

* OpenTelemetry
* Prometheus client libraries

---

## DevOps/SRE Engineers

Responsible for:

* Deploying monitoring stack
* Configuring logging systems
* Managing tracing platforms

Tools include:

* Prometheus
* Grafana
* EFK Stack
* Jaeger

---

# Tools Covered in the Series

## Monitoring

* Prometheus
* Grafana
* AlertManager

## Logging

* Elasticsearch
* Fluent Bit
* Kibana (EFK Stack)

## Tracing

* Jaeger
* OpenTelemetry

## Advanced

* eBPF

---

# Key Takeaways

* Observability helps understand system health deeply.
* Three pillars:

  * Metrics
  * Logs
  * Traces
* Monitoring is only one part of observability.
* Observability is essential in modern Kubernetes/cloud systems.
* Developers and DevOps engineers must work together.
* Strong observability improves reliability, debugging, and customer experience.
