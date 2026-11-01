In this lab, you will add visualizations to your dashboard using the web logs sample data.

**Introduction**

In this challenge, you'll create a bar chart visualization showing HTTP response codes broken down by destination country. You'll learn to use the **Breakdown** feature to add a second dimension to your chart and customize the appearance with colors and labels.


**Key Terms**

<img width="1717" height="400" alt="image" src="https://github.com/user-attachments/assets/e6195418-5b5e-448c-9bfa-cb81068a012f" />

**Creating a Bar Vertical Unstacked visualization**

1. Open the Logs Overview dashboard.
2. Set the time filter to the Last 7 days.
3. Click the Edit button. Note: If the dashboard is already in Edit mode, you'll see a Save button instead.
4. Click Create visualization. This will open the Lens editor by default.

<img width="3424" height="206" alt="image" src="https://github.com/user-attachments/assets/fd4fc06e-689a-4ec7-942b-17dea785793b" />

5. Make sure the **Dataview** is set to `Kibana Sample Data Logs`. Drag the `response.keyword` field to the workspace. This gives you a `Bar vertical stacked` graph.

<img width="3424" height="1630" alt="image" src="https://github.com/user-attachments/assets/eac12ecd-d9e7-455e-ae09-b1b6142e50ad" />

🔍 Why this matters

When you drag a keyword field to the workspace, Lens automatically creates a bar chart showing the count of documents for each unique value. Lens chooses "Bar vertical stacked" as the default because it's a common starting point for categorical data.

6. Drag the `geo.dest` field to `Breakdown` in the layer pane.

<img width="818" height="882" alt="image" src="https://github.com/user-attachments/assets/ab095323-265e-48c6-b74b-c6d52679819f" />

Your graph should resemble the following (the actual numbers may vary):

<img width="1962" height="1348" alt="image" src="https://github.com/user-attachments/assets/698b12cc-edaf-4cea-a7d5-bd0978db6620" />

🔍 Why this matters

Adding a field to Breakdown segments each bar by the values of that field. In this case, each response code bar is now broken down by destination country, letting you see which countries contribute to each response code.

7. Let's adjust the appearance of the graph to create a better view. In the Layer pane, click Top 3 values of geo.dest. Under Appearance, change Color palette to "Elastic classic".

<img width="343" height="367" alt="image" src="https://github.com/user-attachments/assets/6073058c-eb1e-4628-b487-03715982e46d" />

8. Using the dropdown in the layer pane that is set to `Stacked`, change it to `Unstacked`.

<img width="856" height="724" alt="image" src="https://github.com/user-attachments/assets/b378b404-62f1-4936-b875-bdb728250357" />

🔍 Why this matters

Unstacked bars are placed side by side rather than on top of each other. This makes it easier to compare the exact values between categories (destination countries) for each response code, since all bars start from the same baseline.

9. In the `Layer pane`, click `Count of records`.

This opens options for the vertical axis. There are a number of quick formulas available.

<img width="834" height="1000" alt="image" src="https://github.com/user-attachments/assets/c2457cc1-5b59-492f-98b6-b9e4a7d58cbe" />

10. Change the Name to be Number of Requests. Close the panel.

<img width="716" height="1948" alt="image" src="https://github.com/user-attachments/assets/6fc3edc1-e301-438b-bc8f-827022b47270" />

11. Click `Top 5 values of response.keyword`. In the `Layer pane`, change the `Name` to "Response Code", and close the panel.

<img width="774" height="342" alt="image" src="https://github.com/user-attachments/assets/737baafd-20f3-4873-a618-d01b5e7052d1" />

🔍 Why this matters

Renaming axes and series makes visualizations easier to understand for viewers who may not be familiar with the underlying field names. "Response Code" is clearer than "Top 5 values of response.keyword".

12. Click `Save and return` to return to the dashboard. Your visualization should resemble the following (the actual numbers may vary):

<img width="1714" height="848" alt="image" src="https://github.com/user-attachments/assets/b83ba899-cc9d-4bc6-9285-a1678780cfbc" />

13. Save the dashboard.
