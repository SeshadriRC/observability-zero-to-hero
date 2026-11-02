**Introduction**

In this lesson, you'll create a URL drilldown that navigates users from a dashboard visualization to an external website. Drilldowns transform static visualizations into interactive entry points, allowing users to seamlessly move from high-level dashboards to detailed investigation in external systems.

**Key Terms**

<img width="1601" height="438" alt="image" src="https://github.com/user-attachments/assets/70d11a9f-9e4f-474c-a395-66418c478398" />

**Creating a drilldown**

1. Click `Edit` on the Logs Overview Dashboard.

2. Navigate to the pie chart that displays operating systems in the `Logs Overview` Dashboard.

3. Click on the panel options and select `Create drilldown`.

<img width="1462" height="934" alt="image" src="https://github.com/user-attachments/assets/9d4365ad-c311-452c-be43-b8250ab352ff" />

4. Select **Go to URL**.

- For the Name enter `Google search`.
- Select the `Single click` trigger.
- Set the **Enter URL** value to: `https://www.google.com/search?q={{event.value}}`

<img width="1770" height="1832" alt="image" src="https://github.com/user-attachments/assets/bf759f56-78c0-4c5e-bbf6-f980c630f892" />

🔍 Why this matters

The `{{event.value}}` is a Handlebars expression that Kibana replaces with the actual value from the clicked element. When a user clicks a pie slice for "ios", Kibana substitutes that value into the URL, resulting in `https://www.google.com/search?q=ios`. This makes drilldowns dynamic and context-aware.

💡 Best practice

The Single click trigger executes the drilldown immediately when users click. For panels with multiple drilldowns, consider using the Context menu trigger instead, which displays a menu of available actions.

5. Click on `Create drilldown`
6. Test the drilldown by clicking on a pie slice value and selecting `Google search`. This will open a new window with a google search for that operating system.

<img width="1482" height="860" alt="image" src="https://github.com/user-attachments/assets/1294381e-2c4f-47d5-8fdc-77bdbe677a79" />

<img width="2968" height="2622" alt="image" src="https://github.com/user-attachments/assets/a20e283d-b329-44d4-a53c-fc9faab6d3fc" />

7. Save the dashboard.

**Conclusion**

In this lesson, you created a URL drilldown that uses dynamic values from a visualization to navigate users to an external site for deeper investigation.
