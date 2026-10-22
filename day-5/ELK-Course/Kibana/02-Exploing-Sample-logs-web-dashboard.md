**Introduction**

In this lesson, you'll explore an interactive dashboard built from the Sample web logs data. You'll learn how to open a dashboard, set time filters, and use click-and-drag interactions to filter data across all visualizations simultaneously. These skills are fundamental to exploring any dataset in Kibana.

<img width="1330" height="477" alt="image" src="https://github.com/user-attachments/assets/02b26c20-1633-4b12-8858-c8010b73b11f" />

**Exploring the sample logs dashboard**

1. On the Sample data page, select the drop down menu for Other sample data sets on the bottom of the page. Next, locate the Sample web logs you previously added and select View data. You will see a drop down menu with several options available. Click on the Dashboard icon.

<img width="637" height="492" alt="image" src="https://github.com/user-attachments/assets/6d795f6a-0487-4a64-b5e4-612442a4383f" />

2. Before you start exploring the content of the dashboard, take a look at your time filter. You can find the time filter on the top right corner of the workspace. The time filter should automatically be set to last 7 days. If it is not, select the calendar symbol to set the time filter to Last 7 days on the list of Commonly used date ranges.

🔍 Why this matters

The time filter is one of the most important controls in Kibana. Since the sample data is generated relative to the current date, setting the correct time range ensures you see data in all visualizations. In production, you'll frequently adjust time filters to investigate specific incidents or compare time periods.

<img width="1917" height="1055" alt="image" src="https://github.com/user-attachments/assets/33890ac1-e934-435e-868b-a2a4a19d9deb" />

3. Your dashboard contains different types of visualizations. Some are interactive. Hover your mouse over all the panels to see which ones react to your action and which ones don't. Depending on which editor was used to build the visualization, they may have other available mouse actions. Scroll down to the bar chart titled Logs Bytes distribution. Hovering over any bar will show a tooltip with the relevant values for that bar.

<img width="1456" height="487" alt="image" src="https://github.com/user-attachments/assets/f09d97d9-d553-4c1c-9188-5ea9de54413c" />

4. Select and drag across several bars.

<img width="1506" height="784" alt="image" src="https://github.com/user-attachments/assets/c19b880f-8164-4a43-b34c-e75e4ebac9a6" />

When you release the mouse button, you'll notice the bar chart has changed and so have all the other visualizations. Additionally, a filter has been applied based on your selection in the Logs Bytes distribution visualization.

<img width="1481" height="492" alt="image" src="https://github.com/user-attachments/assets/b614710f-6449-4af7-b0b9-e77dfd3d401f" />

🔍 Why this matters

Select-and-drag filtering is a powerful way to drill down into your data. When you select a range in one visualization, Kibana creates a filter that applies to the entire dashboard. This lets you quickly investigate subsets of your data. For example, this will allow isolating high-traffic periods or specific time ranges without writing queries manually. Understanding how filters appear and can be managed is essential for dashboard exploration. Filters persist until you remove them, which can sometimes cause confusion if you forget one is active. Always check the filter bar when your visualizations show unexpected results

<img width="1467" height="792" alt="image" src="https://github.com/user-attachments/assets/0cbd699a-a674-46c4-a66e-635d7290fbc7" />

- Filter removed

<img width="1440" height="851" alt="image" src="https://github.com/user-attachments/assets/d2320cca-637e-4dff-8e67-b3e354f3b990" />
