Troubleshooting with APM Lab

In this lab you will practice using Elastic APM to diagnose and resolve a production issue where customers cannot add products to their shopping cart.

Lab Challenge: APM Incident Response

You've learned how to use APM for troubleshooting by detecting issues, investigating root causes, and applying fixes. Now practice these skills on a live production issue affecting customer shopping functionality.

Your Mission

Customer support reports that users cannot add products to their shopping cart. Use APM investigation techniques to identify the failing service, determine the root cause, and restore shopping functionality.

**Goal:** Complete the full troubleshooting workflow from problem reproduction through verified resolution using the APM investigation skills from the tutorial.

Requirements

Complete these troubleshooting tasks:

✅ Reproduce the customer issue by attempting to add a product to your cart

✅ Identify which service is experiencing transaction failures

✅ Determine the specific transaction that is failing

✅ Find the root cause by examining trace details and failing spans

✅ Resolve the issue and verify normal shopping functionality is restored

Hints to Guide You

**Getting Started:**

- Test the shopping experience first in the OTel demo tab
  
- Navigate to Observability > APM > Services in Kibana to begin investigation
  
- Look for services with elevated failure rates

**Task 1: Problem Reproduction**

- Try adding any telescope or product to your cart in the demo store
- Note the specific behavior you observe when the add-to-cart action fails
- This confirms the customer reports and gives you the user impact context

**Task 2: Service Investigation**

- Sort the Services overview by failed transaction rate to identify problematic services
- Look for services with non-zero failure rates that correlate with shopping functionality
- Focus on services involved in product catalog, cart, or checkout operations

**Task 3: Transaction Analysis**

- Navigate into the failing service and examine its Transactions tab
- Identify which specific transaction operation is failing
- Look for transactions related to cart operations or product interactions

**Task 4: Root Cause Analysis**

- Examine trace samples for the failing transaction
- Click on failing spans to view detailed error information
- Look for external service calls or database operations that might be failing
- Use span metadata to understand exactly what operation is causing the failure

**Task 5: Issue Resolution**

- Based on your root cause findings, check if there are configuration options available
- Look in the Feature Flags tab for any settings that might resolve the issue
- Apply the fix and wait for the changes to take effect

**Navigation Reminders:**

- Use failure rate sorting to quickly identify problem services
- Click service names to drill down into transaction details
- Examine trace samples to see the complete request flow
- Click failing spans to see detailed error information

Success Criteria

You've successfully completed the lab when you can:

- Describe the customer impact and which shopping action fails
- Name the specific service and transaction experiencing failures
- Explain the root cause based on span analysis
- Demonstrate that shopping functionality is restored after your fix

**Ready to troubleshoot?** Start by reproducing the issue, then systematically investigate using APM data.

💡 Need help? Click here for example troubleshooting results

Example Troubleshooting Results:

**Task 1: Problem Reproduction**

Navigate to the store and attempt to buy one or two products. When we try to select the first product, a telescope, the operation fails:

1. Navigate to the OTel store and click Go Shopping. From the Hot Products page, select the first telescope. Right away we see that the product lookup has failed:

<img width="1702" height="1014" alt="image" src="https://github.com/user-attachments/assets/a0c4c4c3-50ed-4131-9588-260f2124aa0f" />

Interestingly enough, the Add to Cart button still appears. Don't try to click it yet!

2. Test to see if all product lookups fail. When we add the second product, also a telescope, the lookup succeeds!

<img width="1652" height="1356" alt="image" src="https://github.com/user-attachments/assets/1d543634-5f3a-4244-aa07-1253e7f2fdc7" />

This could be a problem with specific products or it could be an intermittent service failure.

3. Confirm that you can purchase the second telescope to explore any effects on the checkout flow on the products not affected by the product catalog apparent problems:

<img width="1578" height="1410" alt="image" src="https://github.com/user-attachments/assets/8e1b3b29-a0ec-4c06-ab32-4b2d4ee67123" />

We can indeed purchase the second telescope.

4. This would be a good time to see if the sporadic catalog service failures are impacting checkout, the store's most critical function. Navigate to the services overview in APM and examine the transaction failure rate for the checkout service. Make a note of the througput, as well.

<img width="1764" height="474" alt="image" src="https://github.com/user-attachments/assets/85b830e9-1b9a-413d-a91f-712044da6069" />

We can see that checkouts are failing, but not necessarily all of them.

5. What about the Add to Cart button for the first telescope? Go shopping again and select the first telescope, but this time click Add to Cart, something that at least some customers are likely to do:

<img width="1576" height="912" alt="image" src="https://github.com/user-attachments/assets/ead64b8c-8206-4210-9eae-1126fda2c37c" />

The click takes us to an empty shopping cart!

6. Now try to see if you can still buy the second telescope or if the cart and checkout functionality has been impacted by the `Add to Cart` button click.

<img width="1568" height="812" alt="image" src="https://github.com/user-attachments/assets/173cc764-af08-4096-8242-f863b943f0c6" />

It's not possible to add the second telescope to the cart any more. The cart now appears to be broken.

- Customer impact: Products cannot consistently be selected although the Add to Cart button still appears.

- Specific behavior: Add to cart, which worked fine on some products, now always fails after adding a broken product to the cart.

- Business impact: Customers cannot make purchases, directly affecting revenue.

**Task 2: Service Investigation**

Time to find the problem! Navigate to the services overview and look for services with failing transactions or throughput. Sorting the services in descending order by transaction-failure rate We can see elevated failure rates for `product-catalog` and `checkout`. We know that we can no longer buy any product at all, but other customers might still be able to check out.

<img width="1722" height="340" alt="image" src="https://github.com/user-attachments/assets/0c4f6205-60fc-411e-8fbf-7c1a073a34e2" />

We note the following:

**Failing service:** Both `checkout` and `product-catalog` are unhealthy.

**What's likely happening:** Product catalog is required for checkout in order to validate products.

**Task 3: Transaction Analysis**

Let's analyze `checkout` transactions to confirm that it is indeed the `product-catalog` service that is causing the issues across the store.

Failing transaction and root cause: Navigate to `checkout`, go to the `Transactions` tab and drill down into the `oteldemo.CheckoutService/PlaceOrder` transaction. Find one of the transactions that fails. The visualization clearly shows that the `oteldemo.ProductCatalogService/GetProduct` span is the root cause of the checkout transaction failure.

<img width="1768" height="882" alt="image" src="https://github.com/user-attachments/assets/eff391a0-0be7-4b00-9313-13ceec42572f" />

Click on the span to see more details:

<img width="1552" height="1496" alt="image" src="https://github.com/user-attachments/assets/9c18eabe-3f84-4940-bb93-28e12c10317a" />

- **Pattern:** Some product lookup operations are failing, preventing product selection, adding to the cart, and checkout.

**Task 4: Root Cause Analysis** In this example (and due to our use of the Otel demo store), the cause could well be an active feature flag generating the chaos. In a real situation, we would have to systematically troubleshoot the product catalog service.

Navigate to the Feature Flags tab in your lab and see if that is the problem.

<img width="1878" height="532" alt="image" src="https://github.com/user-attachments/assets/e839cb7f-9ecd-47a0-beb4-348c8c0e0a32" />

After confirming that the `productCatalogFailure` feature flag is set, we conclude that this is certainly one of the root causes.

**Task 5: Issue Resolution**

Solution: Disable the feature flag to eliminate this potential root case.

<img width="1860" height="542" alt="image" src="https://github.com/user-attachments/assets/3bdadc3d-0706-4c16-8ab0-2c774061b43c" />

- **Verification:** Test adding products to cart again and checking out.
  
- **Success metrics:** After some time, the product catalog failure rate drops back down to 0% and becomes healthy again.

 <img width="1718" height="922" alt="image" src="https://github.com/user-attachments/assets/cba2a956-e726-48f6-adae-59f117583876" />

 Most importantly, customers can now buy from the store! The cart is healthy.

 <img width="2070" height="1444" alt="image" src="https://github.com/user-attachments/assets/c336da9d-4603-476c-a7b1-1494f7b54084" />

And the store makes money.

<img width="1632" height="1474" alt="image" src="https://github.com/user-attachments/assets/3efd9327-95fc-4cf7-b887-22e8d90c511a" />

**Key Learning:** APM's strength in incident response is connecting user impact (can't add to cart) to specific technical failures (product catalog service down) through systematic investigation. The service → transaction → trace → span drill-down pattern provides a reliable methodology for root cause analysis in distributed systems.

**Real-world application:** This workflow mirrors production incident response where you start with customer complaints, use APM to identify failing services, examine traces to find root causes, and verify fixes through both technical metrics and user experience validation.
