**Introduction**

In this lesson, you'll add a second visualization to your dashboard and learn how to change chart types in Lens. You'll create a pie chart showing the distribution of operating systems—a common use case for understanding categorical data composition.

**Prerequisites:** Completed "Adding a Visualization" challenge with a bar chart on your dashboard

<img width="1696" height="340" alt="image" src="https://github.com/user-attachments/assets/28a54269-5ef1-4d79-b155-bc75e3b7f447" />

**Adding and Customizing more Visualizations**

1. After saving your dashboard in the last challenge you may it may not be present when you load into this challenge. Select the Dashboard breadcrumb button at the top of the screen then select the dashboard you saved in the last lesson.

<img width="418" height="247" alt="image" src="https://github.com/user-attachments/assets/6e38cbe2-dff0-44e7-9660-8650e75b2587" />


2. You should now see the visualization you just created earlier at the top of the page. Select the `Edit` button in the top right of the dashboard, then select the large blue Create `visualization` button again.

3. This time drag and drop the field `machine.os.keyword`. You can find the field by scrolling down the field list or (partially) typing in the field name in the search bar.

<img width="825" height="566" alt="image" src="https://github.com/user-attachments/assets/ff812705-782e-4f58-9774-67da718f1649" />

🔍 Why this matters

The `.keyword` suffix indicates this field is indexed for exact matching and aggregations, not full-text search. When visualizing categorical data like operating systems, browsers, or status codes, always use the `.keyword` version of the field. Lens uses this field type to count occurrences of each unique value.

4. Change the visualization to a pie chart. You can change the visualization type using the drop down menu on the top of the pane on the right or select it from the `Suggestions` at the bottom. Click `Save and return` to go back to the dashboard.

 <img width="441" height="650" alt="image" src="https://github.com/user-attachments/assets/71c3951d-62bf-4d34-b70c-dfa8a8bc999c" />

Donut charts and pie charts are useful to visualize distribution of categorical data where you want to show something to be a part of a whole. This donut chart shows the distribution of five Operating Systems used by the visitor to our website. A single stack bar chart can be used as an alternative to a donut or pie chart. Sometimes it may be easier to judge the distribution better in a bar chart simply because length is used instead of area.

💡 Best practice

Pie charts work best with 2-5 categories. When you have more categories, the slices become difficult to distinguish. For data with many categories, consider using a horizontal bar chart instead, which makes comparisons easier and accommodates longer labels.
