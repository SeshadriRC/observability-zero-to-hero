- Use multistep monitors with Playwright journeys to simulate customer interactions with the eCommerce store.

04 - Create a MultiStep Browser Monitor

Monitor Customer Behavior using a Multi-Step Browser Monitor

Browser monitors test what your users actually experience. Let's create a monitor that simulates a customer purchasing a product, building the journey step by step.


1. Create a new browser monitor

a. In Kibana, navigate to Observability > Synthetics > Monitors and click Create Monitor.

<img width="2226" height="1430" alt="image" src="https://github.com/user-attachments/assets/5ed40271-8773-4a31-a410-69f61f28ef29" />

b. Make sure Multistep is selected.

<img width="1696" height="1006" alt="image" src="https://github.com/user-attachments/assets/30e70bd4-683e-4f88-aa1a-ce5c101cee1d" />

c. For monitor details, use the name `otel_ecommerce_store_multistep_monitor` and choose `private_location` from the locations dropdown. If you prefer, increase the frequency to Every 5 minutes for this tutorial.

<img width="1684" height="1110" alt="image" src="https://github.com/user-attachments/assets/068eecf8-96a2-4f87-bdc5-f3a4da87b0b9" />

2. Access the store homepage

We are going to create the user journey in steps. Start by copying this initial script into the Script editor in the Add script section of the page.

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});
```

Set the store `endpoint` parameter to `frontend-proxy-nodeport`.

```bash
{"endpoint": "frontend-proxy-nodeport"}
```

<img width="2474" height="1198" alt="image" src="https://github.com/user-attachments/assets/70690b55-b228-47db-8475-b00e888f902a" />

Click to learn more about journey structure

- page → Browser page object for interactions
- params → Configuration parameters passed to the monitor
- step() → Individual measured actions within the journey

3. Verify store access

Click Run test at the bottom of the screen to verify the customer can access the store homepage.

<img width="2232" height="960" alt="image" src="https://github.com/user-attachments/assets/50270cbc-5d35-4d89-84f9-e3a8579b7926" />

4. Browse products

Now, let's go shopping! Go back to the script editor and add a step that simulates a customer clicking the Go Shopping button on the store home page. For your convenience, complete journeys will be shown, not just the last step.

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});

step('Find the shopping button', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await page.screenshot({ path: 'before-click.png' });
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
  await page.screenshot({ path: 'after-click.png' });
});
```

Click to learn more about locators

- `page.locator('text=Go Shopping')` → Finds elements by visible text

- `await expect().toBeVisible()` → Waits for element to become visible

- `expect(page.url()).toContain()` → Verifies navigation succeeded

- Text locators are robust and match user perspective

Click `Run test` to confirm customers can navigate from the homepage to the product catalog.

<img width="2212" height="1016" alt="image" src="https://github.com/user-attachments/assets/402dc451-fb66-4617-8b0e-9a820bb16a1b" />

`Customer experience:` Customer successfully navigates to product listings and can see available items.

5. Select a specific product

Next, select the telescope, which happens to be the first product, to simulate a customer choosing an item:

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});

step('Find the shopping button', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await page.screenshot({ path: 'before-click.png' });
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
  await page.screenshot({ path: 'after-click.png' });
});

step('Select product', async () => {
  await page.locator('.sc-bfbc57fa-0').first().waitFor();
  const productCount = await page.locator('.sc-e8cdbf93-0').count();
  await page.locator('.sc-e8cdbf93-0 >> nth=' + 0).click();
});
```

Click to learn more about CSS selectors

`.sc-bfbc57fa-0` → CSS class selector (styled-components auto-generated classes)

`.first()` → Selects the first matching element when multiple exist

`.waitFor()` → Waits for element to be attached to DOM before interaction

`.count()` → Returns number of matching elements (useful for validation)

`>> nth=0` → Elastic Synthetics syntax for selecting by index

⚠️ CSS classes are brittle - use data-testid attributes in production when possible for stability

Run the monitor to confirm customers can view individual product details.

<img width="2226" height="1324" alt="image" src="https://github.com/user-attachments/assets/0b64a4d6-797a-4f75-80dc-7f02eb9230b6" />

Customer experience: Customer can click on products and view detailed information including price, description, and purchase options.

6. Add product to shopping cart

Now simulate a customer adding the product to their cart:\

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});

step('Find the shopping button', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await page.screenshot({ path: 'before-click.png' });
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
  await page.screenshot({ path: 'after-click.png' });
});

step('Select product', async () => {
  await page.locator('.sc-bfbc57fa-0').first().waitFor();
  const productCount = await page.locator('.sc-e8cdbf93-0').count();
  await page.locator('.sc-e8cdbf93-0 >> nth=' + 0).click();
});

step('Add product to cart', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Add To Cart').click()
  ]);
});
```

- Click to learn more about navigation handling

`Promise.all()` → Executes multiple async operations simultaneously

`page.waitForNavigation()` → Waits for page navigation to complete

This pattern handles clicks that trigger page changes

Prevents race conditions between click and navigation

`page` → Browser page object for interactions (same as Playwright)

`params` → Configuration parameters passed to the monitor (for environment flexibility)

`step()` → Individual measured actions within the journey (each step is timed separately)

Steps run sequentially → if one fails, subsequent steps don't execute

Each step gets its own performance metrics and screenshots

Run the monitor to ensure customers can successfully add items to their shopping cart.

<img width="2202" height="1422" alt="image" src="https://github.com/user-attachments/assets/12f1dc13-878c-4171-b1c9-b5c772df575a" />

**Customer experience:** Product is added to cart and customer sees cart contents with correct item and pricing.

7. Complete the purchase

Add the final step where customers complete their order:

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});

step('Find the shopping button', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await page.screenshot({ path: 'before-click.png' });
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
  await page.screenshot({ path: 'after-click.png' });
});

step('Select product', async () => {
  await page.locator('.sc-bfbc57fa-0').first().waitFor();
  const productCount = await page.locator('.sc-e8cdbf93-0').count();
  await page.locator('.sc-e8cdbf93-0 >> nth=' + 0).click();
});

step('Add product to cart', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Add To Cart').click()
  ]);
});

step('Place order', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Place Order').click()
  ]);
  await page.locator('text=Done').click();
  expect(await page.isVisible('text=Done')).toBeTruthy();
});
```

Click to learn more about verification


page.isVisible() → Returns boolean for element visibility (non-blocking check)

.toBeTruthy() → Jest-style assertion for boolean values

Final verification confirms the business process completed successfully

Multiple verification strategies help catch different failure types:

- Visibility checks → UI rendering issues
- URL verification → Navigation problems
- Text content → Data/content issues

Always verify the most critical business outcome (order completion)

Click `Run test` to execute the complete customer purchase journey from homepage to order completion.

<img width="2206" height="1388" alt="image" src="https://github.com/user-attachments/assets/ed0c29a2-d308-44db-a18b-57203fc556d8" />

Customer experience: Customer successfully completes entire purchase process and receives order confirmation.

Click Create monitor to complete the step.

8. Review purchase journey performance

Examine how long each part of the customer journey takes and identify any bottlenecks in the shopping experience.

What to analyze:

- Step duration trends (getting slower over time?)

- Network request patterns (too many requests?)

- Screenshot evidence of UI changes or errors

- Failed requests or slow API calls

a. From the Monitors > Overview tab, click in the three dots that appear when you hover over the otel_ecommerce_store_multistep_monitor tile and then click Go to monitor.

<img width="596" height="760" alt="image" src="https://github.com/user-attachments/assets/70631fd5-cd47-4190-89f2-52d7a3e2a56c" />

b. Scroll down to view the last test run and click on View test run.

<img width="936" height="986" alt="image" src="https://github.com/user-attachments/assets/24c0b85e-1481-4d9c-9ac6-c5811dbac465" />

The `Test run details` will show you a high-level view of each step, as well as display the screenshot captured during the journey.

<img width="1688" height="1202" alt="image" src="https://github.com/user-attachments/assets/bfb33917-e686-4f00-994d-5a7085bee5bd" />

c. Click on View performance breakdown.

For each step, you can view its timing breakdown.

<img width="1764" height="1410" alt="image" src="https://github.com/user-attachments/assets/9e4cbd56-c997-4d1e-be6e-c86a0b777ceb" />

Scroll down to view the step's metrics.

<img width="1678" height="826" alt="image" src="https://github.com/user-attachments/assets/d85304c5-8eb5-40c3-949b-bd051e6cc8ca" />

Scroll down again to view the network requests that occurred during the execution of the step.

<img width="1732" height="1184" alt="image" src="https://github.com/user-attachments/assets/d4942cfb-c891-4899-a433-f8974a14e88e" />


9. Simulate a payment service outage

Now we are going to proactively simulate a payment service outage and use the synthetics journey you just created to observe the experience of a customer trying to purchase a product.

a. Navigate to the `Feature Flags` tab

b. Change the paymentUnreachable flag state from `off` to `on`.

<img width="1904" height="1580" alt="image" src="https://github.com/user-attachments/assets/3e9c7442-919e-4c2a-baaa-cc2173018369" />

c. Click Save to activate the flag.

d. Return to your monitor and run a new test.

You will see that the journey fails to complete.

<img width="950" height="1294" alt="image" src="https://github.com/user-attachments/assets/5776109a-9067-4a16-95d7-cad1c373ed7d" />

**Customer impact:** Customers can browse and add products to cart, but cannot complete their purchase.

10. Restore payment functionality

a. Re-enable the Payment Service flag
b. Run the monitor to verify customers can complete purchases again

You should see a successful test run once again.

Journey insight: Building customer journeys incrementally helps you understand exactly where the customer experience might break and how it impacts business outcomes.

Next: In the next lab, you will practice what you learned in this tutorial and create additional customer journeys to monitor different shopping scenarios like product search or account management.
