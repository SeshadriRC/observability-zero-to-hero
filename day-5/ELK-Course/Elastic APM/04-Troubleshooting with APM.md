Troubleshooting with APM

Learn how to detect application issues in real-time and investigate root causes using Elastic APM data, traditional investigation techniques, and AI-assisted troubleshooting.

04 - Troubleshooting with APM

Troubleshooting with APM

In this tutorial, you'll learn how to detect and investigate application problems in production environments.

1. **Establish Baseline Health**

First, let's verify that our ecommerce application is running normally. Go to Observability > APM > Services in Kibana

Observe the current application health indicators and make sure they are in a healthy state:

✅ **Failed transaction rate:** Close to 0% across all application services

✅ **Service latencies:** Within expected ranges (for example, checkout ~1-2s, payment ~0.5-1s)

✅ **Active throughput:** Services processing requests (not "N/A")

✅ **Service map nodes:** Green rings or no warning indicators on service nodes

Note

You might see non-zero failure transaction rates in an application service such as recommendation. However, since your data will likely be different, it would be hard for you to follow this tutorial if we try to troubleshoot such services. Instead, we will induce a failure so your data will be very similar to what you will see in the tutorial.

2. Check the Service Map

Click the **Service Map** tab and verify all service nodes appear healthy. Look for:

- **Green rings:** Around service nodes indicating healthy status
- **No red rings or notification badges:** Which would indicate alerts or anomalies
- **Clean service node indicators:** All services should show normal status
  
**Key Point:** This baseline represents normal application performance. When issues occur, you'll see changes in service node indicators (colored rings, notification badges) rather than connection colors.

3. Make the payment service unreachable

In your lab, navigate to the Feature Flags tab. Change the state of the paymentUnreachable flag from `off` to `on`.

<img width="2072" height="1434" alt="image" src="https://github.com/user-attachments/assets/417b6233-978f-40b3-bb40-f990ee84af4f" />

Click `Save` and wait a few minutes for the chaos induced by the flag to affect store operations.

4. Attempt to buy something from the store

In the meantime, navigate to the OTel demo store and try to buy something. Try buying, for example the first Hot Product, a telescope. You will be able to go as far as adding your telescope to the cart, but you will not be able to place the order.

**Signs of trouble:** Customers will begin to complain that they cannot complete their purchases. In another tutorial, we will set up alerts that will send notifications when services are experiencing abnormal transaction failure rates. Whichever way we find out there is an issue with the store, we have to find the root cause and find a solution.

5. Check the health of the system

Sort the services Overview tab in descending order by transaction failure rate and look for a service with failing transactions.

Note

Your data, will of course, be different, but you should see similar a similar pattern caused by the chaos introduced by the paymentUnreachable feature flag.

Clearly, the `checkout` service, with a `95%` failure rate, in this example, is having significant issues. We know why, since we caused it, but, in general, users would see only the symptoms. They might just be alerted to the fact that customers are having problems checking out. Let's follow a systematic workflow and find the cause.

<img width="1822" height="1340" alt="image" src="https://github.com/user-attachments/assets/269fd95e-5cbc-4044-ad28-71cb7cd99f4c" />

6. Troubleshoot the checkout service

Drill-down into `checkout` and navigate to the Transactions tab. We know some of the transactions are failing, let's see if we can find out exactly where the failure occurs.

<img width="1920" height="1472" alt="image" src="https://github.com/user-attachments/assets/2c1ea568-ab17-4377-8d3d-d00491c51cc2" />

Scroll down and you should see one transaction: `oteldemo.CheckoutService/PlaceOrder`. We are aware that this is exactly what customers cannot do currently. Click on it to examine exactly what goes on in the application during checkout

7. Examine the Gantt chart

You can immediately see that the checkout service is now failing practically all the time -- if you wait long enough, it will fail 100% of the time!

<img width="1836" height="592" alt="image" src="https://github.com/user-attachments/assets/e9e01572-70ee-49d9-b0eb-71fbc23b2d36" />

Scroll down to view trace samples. Navigate until you find a failing transaction (which should be very easy):

<img width="1954" height="1574" alt="image" src="https://github.com/user-attachments/assets/60df3550-b71b-4008-a419-d959f351a371" />

At the bottom of the Gannt chart, you can see the span where the transaction failed.

8. Get the details on the span that fails

Click the failing span to see its details.

<img width="1786" height="1002" alt="image" src="https://github.com/user-attachments/assets/3507ab38-eedc-4162-92e3-6ed21fffca84" />

We can see that it is indeed the payment service span `oteldemo.PaymentService/Charge` that is failing!

9. Resolve the payment service outage

In a real-world scenario, we would need to perform careful and systematic troubleshooting of the payment service problems. However, for this tutorial all you have to do is turn off the feature flag that made the payment service unreachable.

<img width="1860" height="1612" alt="image" src="https://github.com/user-attachments/assets/173ffde1-b8b2-44e3-a991-550bb8453f7c" />

Wait a few minutes and you should start seeing the failure rate for checkout go down.

<img width="1746" height="512" alt="image" src="https://github.com/user-attachments/assets/83b1842d-2610-4c0c-b177-766ab64a866d" />

10. Attempt the purchase again

More importantly, customers can now buy telescopes and other merchandise from the store!

<img width="2008" height="1412" alt="image" src="https://github.com/user-attachments/assets/3482a6c7-2a81-4527-9d3a-aa50179e76d9" />

Congratulations! You've successfully completed a full APM troubleshooting workflow, demonstrating how Site Reliability Engineers detect, investigate, and resolve production issues.

What You've Learned

In this tutorial, you experienced the complete APM troubleshooting methodology:

**Detection:** Used service health indicators to identify the failing checkout service with 95% transaction failures, directly correlating to customer impact.

**Investigation:** Drilled down through multiple APM data layers:

- Service overview to identify the problematic service

- Transaction analysis to isolate the failing operation `(PlaceOrder)`

- Trace examination to understand the complete request flow

- Span details to pinpoint the exact failure point `(PaymentService/Charge)`

**Resolution:** Applied the fix (disabled the chaos flag) and verified both technical recovery (failure rates dropping) and business impact (customers can complete purchases).

**Key Insight:** APM's strength lies in connecting high-level business impact (customers can't buy telescopes) to specific technical failures (payment service unreachable) through distributed traces. This request-centric view enables rapid root cause identification in complex microservices architectures.

You now understand how APM transforms incident response from guesswork into systematic investigation, reducing mean time to resolution and improving customer experience.
