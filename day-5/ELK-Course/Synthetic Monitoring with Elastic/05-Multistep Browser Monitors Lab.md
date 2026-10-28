In this lab you will practice creating user journeys that simulate customer behavior using multistep browser monitors.

05 - MultiStep Browser Monitor - Lab

Lab Challenge: Create a Customer Journey Monitor

You've learned how to create multi-step browser monitors that simulate real user interactions. Now it's time to apply that knowledge and build a monitor that captures a more complex customer shopping experience.

Your Mission
Create a multi-step browser monitor that simulates a realistic customer shopping journey where someone changes their mind during the purchase process.

Customer Journey: A customer visits the eCommerce store, browses products, adds a telescope to their cart, but then changes their mind and empties the cart. They continue shopping and ultimately purchase binoculars instead.

Requirements

Your monitor should:

✅ Successfully navigate through the complete customer journey
✅ Handle cart management (adding and emptying items)
✅ Complete with a successful binoculars purchase
✅ Run without errors and capture screenshots at key moments

**Hints to Guide You**

Getting Started:

Create a new Multistep browser monitor

Name it `customer_journey_lab_monitor`

Use the `private_location` you created earlier

Set the endpoint parameter: `{"endpoint": "frontend-proxy-nodeport"}`

**Journey Building Tips:**

- Start by loading the homepage (similar to the tutorial)

- Navigate to the product catalog using the "Go Shopping" button
- The telescope is the first product (index 0) in the product listing
- After adding the telescope to cart, look for an "Empty Cart" button
- Use "Continue Shopping" to get back to browsing
- Click "Go Shopping" again to return to the product catalog
- The binoculars are the 5th product (index 4) in the listing
- Complete the purchase flow with "Place Order" and "Done"
  
**Technical Reminders:**

- Use `step('Step name', async () => { ... })` for each major action
- Text locators like `text=Button Name` are robust for finding buttons
- Use `Promise.all()` with `page.waitForNavigation()` for clicks that navigate
- Take screenshots with await `page.screenshot()` to capture key moments
-Test your monitor with Run test before saving

**Success Criteria**

When your monitor runs successfully, you should see:

- Customer starts on homepage
- Adds telescope to cart
- Empties the cart
- Continues shopping and returns to products
- Adds binoculars to cart
- Completes the order

**Ready to start?** Create your monitor and build the customer journey step by step. Remember, you can always run tests to debug each step as you build it.

💡 Need help? Click here for the complete solution

```bash
step('Load homepage', async () => {
  await page.goto('http://' + params.endpoint + ':8080/');
});

step('Find the shopping button', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
});

step('Select telescope', async () => {
  await page.locator('.sc-bfbc57fa-0').first().waitFor();
  await page.locator('.sc-e8cdbf93-0 >> nth=' + 0).click();
});

step('Add telescope to cart', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Add To Cart').click()
  ]);
});

step('Empty the cart', async () => {
  const emptyCartButton = page.locator('text=Empty Cart');
  await expect(emptyCartButton).toBeVisible();
  await emptyCartButton.click();
  await page.screenshot();
});

step('Continue shopping from cart', async () => {
  const continueShoppingButton = page.locator('text=Continue Shopping');
  await expect(continueShoppingButton).toBeVisible();
  await continueShoppingButton.click();
  await page.screenshot();
});

step('Go shopping again', async () => {
  const shopButton = page.locator('text=Go Shopping');
  await expect(shopButton).toBeVisible();
  await shopButton.click();
  expect(page.url()).toContain('#hot-products');
  await page.screenshot();
});

step('Select binoculars', async () => {
  await page.locator('.sc-bfbc57fa-0').first().waitFor();
  await page.locator('.sc-e8cdbf93-0 >> nth=' + 4).click();
  await page.screenshot();
});

step('Add binoculars to cart', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Add To Cart').click()
  ]);
  await page.screenshot();
});

step('Place order', async () => {
  await Promise.all([
    page.waitForNavigation(),
    page.locator('text=Place Order').click()
  ]);
  await page.locator('text=Done').click();
  await page.screenshot();
});

```

**Parameters:**

```bash
{"endpoint": "frontend-proxy-nodeport"}
```
