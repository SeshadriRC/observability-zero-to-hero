**Introduction**

In this tutorial, you'll explore a microservices e-commerce application and attempt to understand its architecture using traditional command-line tools. This hands-on experience will help you recognize the observability gap—the difference between what you need to know to troubleshoot distributed systems and what traditional tools can show you.

**Prerequisites: Basic familiarity with Kubernetes and kubectl commands**

<img width="1595" height="396" alt="image" src="https://github.com/user-attachments/assets/8741fc89-b802-45f2-97a2-1fe7038051b2" />

**Investigation without Observability**

1. Explore the Astronomy Shop

You should see the Astronomy Shop e-commerce store home page in your lab environment.

<img width="1258" height="815" alt="image" src="https://github.com/user-attachments/assets/a62ef2c3-b02a-479d-864b-e6a981040f49" />

Click Go Shopping, browse products, add items to your cart, and complete a checkout. Notice the key features:

Product catalog with images and prices

Currency conversion

Shopping cart

Checkout with payment processing

Order confirmation

2. Discover the microservices architecture

Based on your exploration, which microservices do you think power this application?

In a typical eCommerce store powered by micrroservices, you would have services like these:

<img width="1505" height="475" alt="image" src="https://github.com/user-attachments/assets/57be0bc0-c035-464a-945b-8639dd635453" />

<img width="1595" height="560" alt="image" src="https://github.com/user-attachments/assets/66acb15b-513c-4c14-b69e-d0e3a27afb6b" />

View the running services

The Astronomy Shop is running on a Kubernetes environment where each service runs inside a pod. Run this command to see all services running in the cluster:

bash

copy
kubectl get pods -n default
You should see output similar to this:

bash

copy
NAME                                  READY   STATUS    RESTARTS        AGE
accounting-58cf885b85-h9bpz           1/1     Running   0               2m14s
ad-69dd8c84f6-g9sdn                   1/1     Running   0               2m15s
cart-6ddf7f755-gg47g                  1/1     Running   0               2m15s
checkout-7f746bb87-rpczl              1/1     Running   0               2m15s
currency-5cbcb5b744-xs887             1/1     Running   0               2m13s
email-76c4f6f495-xhtlw                1/1     Running   0               2m15s
frontend-d5bd7bfc9-h4gzl              1/1     Running   0               2m15s
frontend-proxy-79db9fc9bd-z2ndk       1/1     Running   0               2m13s
payment-ff6449546-qrwqz               1/1     Running   0               2m15s
product-catalog-68598f9cf8-l86t6      1/1     Running   0               2m15s
recommendation-79ddd64cc8-7mgg8       1/1     Running   0               2m15s
shipping-56869b5c66-9tq8h             1/1     Running   0               2m14s
...
Notice:

The services match common e-commerce patterns (cart, checkout, payment, shipping, email)
There are also infrastructure services (kafka, valkey-cart for caching)
Each pod has a random suffix added to its name
🔍 Why this matters
While kubectl get pods shows you what's running, it tells you nothing about how these services interact. You can see the checkout service exists, but not that it depends on cart, payment, shipping, email, and currency services to complete an order. Understanding these dependencies is critical for troubleshooting.

Investigate a specific service

Let's look at the checkout service. First, get its full pod name:

bash

copy
kubectl get pods -n default | grep checkout
You'll see something like:

bash

copy
checkout-7f746bb87-rpczl              1/1     Running   0             5m
However, the suffix of the checkout service pod name will likely be different for you. You will need this exact name in the following step.

Now read its logs:

bash

copy
kubectl logs -n default checkout-7f746bb87-rpczl --tail=20
Replace checkout-7f746bb87-rpczl with the actual checkout pod name you found.

You'll see log entries like:

bash

copy
{"message":"PlaceOrder called with...","severity":"info","timestamp":"2025-10-07T21:55:55.431Z"}
{"message":"Charge request sent to payment service","severity":"info","timestamp":"2025-10-07T21:55:55.432Z"}
...
These logs show what the checkout service is doing, but they're isolated to just this one service.

🔍 Why this matters
Try to investigate a user transaction

Imagine a customer reports their checkout failed. To investigate, you need to see what happened across ALL services involved in that checkout (frontend, checkout, cart, payment, shipping, email).

With kubectl, you would need to:

Get logs from the frontend pod
Get logs from the checkout pod
Get logs from the cart pod
Get logs from the payment pod
Get logs from the shipping pod
Get logs from the email pod
Try viewing logs from the payment service:

bash

copy
kubectl get pods -n default | grep payment
bash

copy
kubectl logs -n default <payment-pod-name> --tail=20
Now you have logs from two services, but:

How do you correlate these logs to the same transaction?
How do you know if these logs are even from the same customer's checkout?
What about the other four services involved?
Recognize what's missing

Look at the logs you've collected. You can see individual events, but you're missing critical information:

No correlation - You can't easily connect logs from different services that are part of the same transaction. Each service logs independently, and matching them up requires manual work with timestamps and request IDs.

No metrics - How fast is the checkout service responding? How many requests per minute? What's the error rate? Logs don't tell you these aggregate measurements.

No traces - Which services did this request actually touch? How did they interact with each other? How long did each service take? Where did the time go? You have no visibility into the request flow.

No system overview - Is the problem isolated to one service or affecting the whole system? Which services are healthy and which aren't? You'd need to check each service individually.

This is the observability gap —the difference between what you need to know and what traditional tools can show you.


Conclusion
In this tutorial, you explored the Astronomy Shop, a cloud-native e-commerce application running as a collection of microservices on Kubernetes.

You used kubectl to list running services and view logs from individual pods. While these tools give you access to raw data, they revealed a fundamental limitation: there is no built-in way to connect logs across services, measure aggregate performance, or visualize how a request flows through the system.

This is the observability gap — and it's exactly the problem that modern observability platforms are designed to solve.

Next, you'll see how Elastic Observability closes that gap by automatically correlating logs, metrics, and traces across all your services.

Select Next to continue.
