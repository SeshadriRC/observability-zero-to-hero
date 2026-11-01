**Introduction**

In this challenge, you'll create a heat map visualization showing unique visitors by hour of day and destination country. You'll learn to switch visualization types, configure cell values with unique counts, and apply color palettes to highlight patterns in the data.

<img width="1480" height="440" alt="image" src="https://github.com/user-attachments/assets/a44179c3-021e-43dd-a9cd-c1fc0f1593e4" />

**Adding and customizing more visualizations**

1. Let's create another visualization. Click the `Create visualization` button to open `Lens` again.

2. Drag three fields to the **Layer pane** this time:

- `hour_of_day` to the Horizontal axis
- `clientip` to the Vertical axis
- `geo.dest` to Breakdown

<img width="2294" height="956" alt="image" src="https://github.com/user-attachments/assets/1b256119-38c6-4668-97c7-393a2b87f391" />

This will result in an interesting bell curve display, and the fields will have some default formulas applied.

<img width="1078" height="928" alt="image" src="https://github.com/user-attachments/assets/4ee8e950-aadd-4a68-be12-06260de4599f" />

Notice the warning icon. Click the warning and enable accuracy mode at the bottom.

🔍 Why this matters

When you drag the `clientip` field, `Lens` applies a unique count aggregation because IP addresses are typically counted as distinct visitors rather than summed. The accuracy mode warning appears because unique counts on high-cardinality fields (like IP addresses) use approximations by default. Enabling accuracy mode provides exact counts.

3. Change the type of the graph to a `Heat map` using the dropdown.

<img width="846" height="996" alt="image" src="https://github.com/user-attachments/assets/a1545f65-e054-44ea-9bee-aace0f40987c" />

🔍 Why this matters

A heat map is well-suited for showing patterns across two dimensions (hour and country). The color intensity of each cell makes it easy to spot when and where traffic peaks occur—patterns that would be harder to see in a bar chart with many categories.

4. Let's change some settings to improve the view. First, click hour_of_day under Horizontal axis so that the Display name is simply Hour. Close the panel.

<img width="790" height="770" alt="image" src="https://github.com/user-attachments/assets/7b2239f4-28a0-4abc-80c5-9fabf2e70ac6" />

5. Next, click `Top 3 values of geo.dest`

   - Change the `Name` to `Destination Country`
   - Increase the `Number of values` to `25`
   - Enable Aggregate by this dimension first
   - Open Advanced and disable Group remaining values as "Other"
   - Close the panel

   <img width="762" height="922" alt="image" src="https://github.com/user-attachments/assets/7ad0dea1-a5ed-4492-aaa6-1b3dd9a2d461" />

6. Click `Unique count of clientip` under "Cell value". Change the following:

   - color palette to `Positive`
   - name to `Visitors`
  
  <img width="576" height="750" alt="image" src="https://github.com/user-attachments/assets/e5ad77d7-d40a-4dff-a126-0709fbbdf630" />

🔍 Why this matters

The Positive color palette uses a gradient from light to dark, where darker colors indicate higher values. This makes it intuitive to spot cells with the most visitors. Naming the metric "Visitors" clarifies that you're counting unique IP addresses, not total requests.

7. Save and return to your dashboard, and then Save your dashboard.
8. Click Switch to view mode and then hover over one of the cells in your Heat map to see the labels.

<img width="1520" height="748" alt="image" src="https://github.com/user-attachments/assets/6b4755e0-28a1-4c2b-9791-65bdd3c85d38" />

Summary

In this lab, you added a few more visualizations to your dashboard and customized the appearance of the visualizations.
