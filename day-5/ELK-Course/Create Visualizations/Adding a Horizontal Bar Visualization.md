Introduction

In this challenge, you'll create a horizontal bar chart showing log counts by destination country and operating system. You'll learn to combine multiple fields on a single axis to create a grouped breakdown and configure the number of values displayed.

<img width="1736" height="367" alt="image" src="https://github.com/user-attachments/assets/3cdd1505-99a6-406e-aed0-448eca377af2" />

Adding a Horizontal Bar Visualization

1. Let's create another type of visualization. Click the Create visualization button to open Lens again.

2. Keep Bar as visualization type and make the bar orientation Horizontal.

<img width="1118" height="486" alt="image" src="https://github.com/user-attachments/assets/9fcd7480-95ab-4e51-b681-cddb52942212" />

3. Drag the following fields to the Layer pane:

- `geo.dest` under Vertical axis
- `Records` under Horizontal axis

<img width="3436" height="1438" alt="image" src="https://github.com/user-attachments/assets/973b4429-c8ba-4ca5-bd10-0d48669b7f2c" />

4. Let's improve the visualization. First, click on Count of records in the Horizontal axis section of Layer pane and set the following:

- Change Name to Number of logs
- Change Series color to `#6092C0` (Blue)

Click `Close` when done.

<img width="852" height="1560" alt="image" src="https://github.com/user-attachments/assets/39cb9e14-7445-4e01-9773-4055e0535ec1" />

5. Click `Top 5 values of geo.dest` and set the following:

- Change Name to Destination Country
- Change Number of values to 25
- Open `Advanced` and disable Group remaining values as "Other"

🔍 Why this matters

Increasing the number of values from 5 to 25 shows more destination countries in the chart. Disabling Group remaining values as "Other" removes the catch-all **Other** bar, which can be distracting when you want to focus on the individual countries displayed.

6. Still in the `Vertical Axis` settings, click `Add field` under the `Fields` section and select the `machine.os.keyword` field. It will combine fields in the horizontal bar to show the amount of `Logs` for every `Destination Country` per `Machine OS`. Click Close when done.

<img width="634" height="206" alt="image" src="https://github.com/user-attachments/assets/f8e3bcc6-9bbc-412a-9dd6-322203ead086" />

🔍 Why this matters

Adding a second field to the same axis creates a nested grouping. Each destination country now shows separate bars for each operating system, letting you see the OS distribution within each country without using the Breakdown feature.

7. Your visualization should resemble the visualization below. Click `Save and return`.

<img width="1920" height="1304" alt="image" src="https://github.com/user-attachments/assets/2ed58f03-1269-4c14-b9eb-ac1b2490a105" />

8. Save the dashboard.
