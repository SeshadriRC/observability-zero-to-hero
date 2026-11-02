**Introduction**

In this challenge, you'll add interactive controls to a dashboard that allow viewers to filter data without editing queries or visualizations. Controls are essential for building self-service dashboards where users can explore data on their own, narrowing down to the specific subset they need.

<img width="1621" height="460" alt="image" src="https://github.com/user-attachments/assets/7d3b87e8-8dd5-4733-9483-d7ac974f3d0c" />

1. Open the Logs Overview dashboard.

- Set the timepicker to Last 7 days.
- Click the Edit button. Note: If the dashboard is already in Edit mode, you'll see a Save button instead.

2. Click the Controls dropdown on the dashboard toolbar and then select Add control. This opens the control creation panel.

<img width="1442" height="364" alt="image" src="https://github.com/user-attachments/assets/5ef56a77-7a32-4d2a-9091-84bfcfd2f95b" />

3. There are three types of controls: Options list, Time slider, and Range slider. Let's start by adding an Options list.

- Leave the data view setting as is and select geo.dest in the field list. The control type is automatically selected depending on the field type.
- Set the label to **Destination**
- Click **Save**

<img width="970" height="1768" alt="image" src="https://github.com/user-attachments/assets/95a7797b-b63b-4d49-adc2-a365c7ed6228" />

🔍 Why this matters

The control type is automatically determined by the field's data type: string and IP fields get Options list controls, while numeric fields get Range slider controls. This automatic selection helps ensure you get the most appropriate filter interface for each field.

4. The control will appear now at the top of your dashboard. Click the control and type fr in the value search menu. Notice it updates the list of available entries. Select the only entry provided for the FR destination.

<img width="846" height="570" alt="image" src="https://github.com/user-attachments/assets/e280bb0f-ec71-43a5-ae06-eb4ef3205953" />

🔍 Why this matters

Options list controls support type-ahead search, making it easy for users to find specific values even when there are many options. This is especially helpful for high-cardinality fields like country codes or hostnames.

5. Add a Range slider. Follow the same steps as done previously, but this time use the field bytes, set the label to Bytes, and select the Range slider Control type. Once done, click Save to create a new control.

<img width="970" height="1768" alt="image" src="https://github.com/user-attachments/assets/03736c87-db28-4be0-99b3-4e64f3135b03" />

6. Click the dropdown for the Destination control and click the Clear selections icon to remove the filter.

<img width="840" height="800" alt="image" src="https://github.com/user-attachments/assets/3837af62-72ef-46ce-bad7-194ab9c99d15" />

7. Hover over the far left side of the new **Bytes** control and drag it to the left of the **Destination** control.

<img width="1660" height="268" alt="image" src="https://github.com/user-attachments/assets/97340273-cc80-467c-a3c6-a8fcb15ac6b7" />

🔍 Why this matters

Control order matters when Chain controls is enabled. Controls are applied from left to right, so placing more restrictive filters first can significantly narrow the options shown in subsequent controls.

8. Click the **Controls** dropdown then select **Add time slider** control. Click on the play button to see the evolution of your data during the last week.

<img width="2982" height="320" alt="image" src="https://github.com/user-attachments/assets/0ea499d6-60a4-4ed5-8f5a-c787df10fd4d" />

🔍 Why this matters

The Time slider provides an animated view of how your data changes over time within the global time range. This is useful for spotting trends, anomalies, or patterns that might not be obvious in a static view.

9. Click Save to make sure your controls are maintained in the dashboard.

⚠️ Unsaved controls will be lost

Controls are part of the dashboard configuration. If you navigate away without saving, any controls you added or modified will be lost.

Conclusion

In this lesson, you added dashboard controls that allow users to filter and explore data interactively, improving usability and enabling self-service analysis.
