- In this lab you will practice creating lightweight TCP Ping and ICMP Ping monitors for the OTel Demo eCommerce store.

**Introduction to Synthetics - Lab**

In this lab you will practice what you learned in the Introduction to Synthetics tutorial to create TCP and ICMP monitors for the OTel demo eCommerce store.

1. Create a TCP Ping monitor for the OTel demo eCommerce store.

a. Monitor type: TCP

b. Host: `frontend-proxy-nodeport`

c. Port: `8080`

d. Location: `private_location`

e. Name: `otel_ecommerce_store_tcp_monitor`

f. **Before running a test:** What do you expect this monitor to check differently than the HTTP monitor?

Answer

- The HTTP Ping monitor checks to make sure specific endpoints return the correct status code and display the correct text.

- The TCP Ping monitor checks the services running on your hosts. The TCP Ping monitor checks individual ports to make sure the service is accessible and responding.

- The ICMP Ping monitor checks the availability of your hosts. It uses ICMP (v4 and v6) Echo Requests to check the network reachability of the hosts you are pinging. This will tell you whether the host is available and connected to the network, but doesn't tell you if a service on the host is running or not.

g. Run a test and compare results with your HTTP Ping monitor from the previous challenge.

<img width="1912" height="837" alt="image" src="https://github.com/user-attachments/assets/00f1ab09-b8fe-4c18-8f04-d3a75fb9e5d8" />

<img width="1917" height="842" alt="image" src="https://github.com/user-attachments/assets/ade6c731-606b-46d6-ad7a-7069275470ed" />


2. Create an ICMP Ping monitor to test basic network connectivity.

<img width="1917" height="705" alt="image" src="https://github.com/user-attachments/assets/1958d713-ebef-48e8-9dbe-da2f481c87ea" />

a. Monitor type: **ICMP (Ping)**

b. Host: `frontend-proxy-nodeport` (no port needed)

c. Location: `private_location`

d. Name: `otel_ecommerce_store_ping_monitor`

**Note**

This monitor may fail in Kubernetes environments where ICMP traffic is blocked by network policies, CNI plugins, or container networking configurations. This is normal behavior and demonstrates real-world monitoring challenges in containerized deployments.

e. Run a test and observe results. If the monitor fails while TCP Ping succeeds, this illustrates how different network layers can have different accessibility - a valuable insight for production monitoring strategies.

f. Think about why might ICMP Ping fail while TCP Ping works?

Answer

Containerized applications, like the OTel demo store which uses kubernetes, may block network traffic at some layers of the IP stack and not others. Containerized applications often require different monitoring approaches than traditional infrastructure.

<img width="1917" height="836" alt="image" src="https://github.com/user-attachments/assets/7f4fc7dc-8668-4cf4-99f7-b1aba747d0da" />


3. Compare your monitor types:

<img width="1917" height="835" alt="image" src="https://github.com/user-attachments/assets/68ba7eda-82dd-4c9f-b9f0-a5b0296e01cc" />


- View your HTTP Ping and TCP Ping monitors side by side (both should be passing)

- If your ICMP Ping monitor failed, compare the working monitors with the failed one

- What different information does each provide?

- How do the failure modes teach us about network layer differences?

- Which monitor would detect problems first in different failure scenarios?

Lab Complete!

Excellent work! You've now created multiple types of lightweight monitors and experienced how they provide different perspectives on your application's health - including learning about real-world monitoring constraints.

What you accomplished:

- Created TCP Ping monitors that test port connectivity without HTTP overhead
- Set up ICMP Ping monitors and learned about containerized networking limitations
- Compared monitor types to understand when each is most valuable and why some may fail in certain environments
- Experienced real-world constraints that affect monitoring strategies in production

Key Insights:

- HTTP Ping monitors test full application functionality and response content

- TCP Ping monitors verify service availability and port accessibility

- ICMP Ping monitors detect network-level connectivity issues but may be blocked in containerized environments

- Layered monitoring catches different types of failures when it's technically feasible -- the failure of your ICMP Ping monitor at the same time your TCP Ping monitor succeeded demonstrated how containerized applications can have different levels of access on the IP network stack.
