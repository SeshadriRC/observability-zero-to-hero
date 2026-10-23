

The OTel demo is a webshop running on Kubernetes and it is composed of several different microservices. We just got informed that users are unable to checkout their orders. Let's simulate the issue in this challenge to see what's happening. In the next challenges we will be using Elastic Agent to find the issue.

1. First, click Go Shopping on the OTel demo tab.

<img width="2660" height="1496" alt="image" src="https://github.com/user-attachments/assets/a55ce864-8a2e-40d4-96c5-dab776bcc3e6" />

2. Next, select any one of the products from the list. For example, we will be using the Roof Binoculars.

<img width="2650" height="1492" alt="image" src="https://github.com/user-attachments/assets/069f2ebe-63ab-47c2-a9f6-43e40273d364" />

3. Then, click Add to cart to test the webshop checkout in the next step.

<img width="2324" height="1308" alt="image" src="https://github.com/user-attachments/assets/b99df72f-93be-4ade-951e-a3fc221bcb10" />

4. Finally, scroll down and try to Place Order. Don't worry about payment details, as they are already filled for you.

<img width="1112" height="490" alt="image" src="https://github.com/user-attachments/assets/549887e7-1b78-4131-add2-d70b0f33ee61" />

5. What happens?

Click to see the solution
When you click the Place Order button, nothing happened, so you assume the order didn't go through. It looks like there's a problem when placing the order. In the next step we will set up Elastic Agent to monitor our Kubernetes environment.
