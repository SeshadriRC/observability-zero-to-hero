**Introduction**

In this challenge, you'll save your completed dashboard and learn about the "Store time with dashboard" option. Proper naming and time filter settings ensure your dashboard is easy to find and displays relevant data when reopened.

**Prerequisites:** Completed "Arranging Panels" challenge with an arranged dashboard

<img width="1636" height="356" alt="image" src="https://github.com/user-attachments/assets/d7cddcf0-cd4d-4bc7-883f-5f264938eabe" />

**Saving and Viewing your Dashboard**

1. Once you have chosen a configuration you're happy with Save your dashboard. Dashboard names should be descriptive. Let's call it "Logs Overview". You can include a description if you want. If you want to store the currently set time filter range with your dashboard, enable Store time with dashboard. Take a look at your time filter before you do this so that you save the time range you want. "Last 7 days" is a good time range to use.

Important

In Kibana, Relative Time defines a time window based on the current moment (e.g., "Last 7 Days," "Now-30m"), showing dynamic, shifting data, while Absolute Time sets a fixed, specific start and end date/time (e.g., Jan 1, 2025, to Jan 7, 2025)

<img width="1404" height="1230" alt="image" src="https://github.com/user-attachments/assets/450f3398-1b4d-403c-9270-fe1fd5a5c7ec" />

🔍 Why this matters

The "Store time with dashboard" option determines the initial time range when someone opens your dashboard. Enable it when your dashboard is designed for a specific time window (like "Last 7 days" for operational monitoring). Leave it disabled when users should see whatever time range they were previously using, which is common for investigative dashboards.

💡 Best practice

Use descriptive, searchable names that indicate the dashboard's purpose and data source—for example, "Web Logs - Traffic Overview" rather than just "My Dashboard." This helps colleagues find relevant dashboards and helps you organize dashboards as your collection grows.

2. Finally, click Switch to view mode on the toolbar to see your completed dashboard. If you want to make more changes, just click Edit.

<img width="1043" height="153" alt="image" src="https://github.com/user-attachments/assets/b0de93a5-c86b-47c4-bc75-0e1ae0b4a89d" />

🔍 Why this matters

View mode shows the dashboard as end users will see it, without editing controls cluttering the interface. Always preview your dashboard in view mode before sharing it to ensure the layout looks correct and all visualizations display properly.

Summary
In this lab, you created a simple dashboard with two charts and one markdown visualization.
