**Introduction**

In this lesson, you'll create your first dashboard and add a visualization using the Lens editor. You'll learn the essential workflow of selecting a data view, setting a time filter, and dragging fields to build a chart—skills you'll use every time you create visualizations in Kibana.

Prerequisites: Completed "Introduction to Kibana" track with sample data loaded

<img width="1721" height="571" alt="image" src="https://github.com/user-attachments/assets/975b0003-f323-4a60-9880-367b25b1dbbd" />

Adding Your First Visualization

1. In Kibana, open the Dashboard editor by selecting the hamburger menu ( ☰ ) then under the Analytics section select Dashboard, then select Create dashboard.

<img width="282" height="495" alt="image" src="https://github.com/user-attachments/assets/8046c179-07f8-4d57-a351-a0471a884b21" />

2. Select either Create visualization button.

<img width="863" height="698" alt="image" src="https://github.com/user-attachments/assets/864aa1e3-c8e3-42b8-8085-5f69c6eb514f" />

3. This will take you to the Lens visualization editor.
4. Before you do anything in Lens make sure you have selected the correct data view and time filter range. If not already specified, select the `Kibana Sample Data Logs` data view and set the time filter to `Last 7 days`.

<img width="1046" height="417" alt="image" src="https://github.com/user-attachments/assets/ff31e4d8-71f7-4006-8cd3-178942fac44b" />

Warning

Always verify data view and time filter first before moving forward.

🔍 Why this matters

A common mistake is building a visualization before checking these settings. If you select the wrong data view, you'll be working with different data than expected. If your time filter excludes the relevant time range, your visualization will appear empty. Always verify these two settings before dragging any fields.

5. Drag and drop the `Records` field (from the fields list on the left) into the workspace (at the center).

<img width="1070" height="709" alt="image" src="https://github.com/user-attachments/assets/df7f9715-674d-40fe-a33e-e8c238460a0b" />

🔍 Why this matters

The Records field counts documents over time, making it perfect for seeing traffic patterns, event volumes, or activity trends. Lens automatically creates a time-based bar chart when you drop Records into the workspace, grouping counts by time intervals. This is often the first visualization you'll create when exploring any time-series dataset.

6. You should now see a vertical bar chart. Click `Save and return` to return to the dashboard space.

<img width="1100" height="630" alt="image" src="https://github.com/user-attachments/assets/29c32424-a1fa-4e69-8bde-0a4889c77417" />

7. You should now be in the `Dashboards` workspace and your visualization should be available. You have now created your first visualization and dashboard.

<img width="1117" height="766" alt="image" src="https://github.com/user-attachments/assets/84b64564-0c5c-4889-9787-9b32e1267901" />

8. Select `Save` in the top right corner and give your dashboard a name. After naming the dashboard select `Save` again to finalize this process.

<img width="1054" height="718" alt="image" src="https://github.com/user-attachments/assets/9781910f-68ab-4de6-a0bd-8a739971467b" />

