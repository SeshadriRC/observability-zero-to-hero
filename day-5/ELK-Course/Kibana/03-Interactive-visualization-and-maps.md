**Introduction**

In this lesson, you'll learn about two visualization editors in Kibana: Lens and Maps. You'll use the Maps visualization to create a geographic filter, demonstrating how spatial data can be explored interactively. These editors are essential tools you'll use throughout your work with the Elastic Stack.


<img width="1020" height="416" alt="image" src="https://github.com/user-attachments/assets/3fd01d9c-ff28-4bd7-9a4c-48f9eba915cb" />


**Interactive Visualizations and Maps**

1. The Logs Bytes distribution bar chart was created through a visualization editor called Lens.

🔍 Why this matters

Lens is the recommended visualization editor for most use cases in Kibana. It uses a drag-and-drop interface that suggests visualization types based on your data, making it accessible for beginners while remaining powerful enough for complex analyses. Understanding which editor created a visualization helps you know how to modify it.

2. Similarly, we can change the view of the data set in other ways. Go to the map visualization titled Logs Total Requests and Bytes. Click the tools (wrench icon) icon and select Draw distance to filter data.

<img width="633" height="728" alt="image" src="https://github.com/user-attachments/assets/65f932e1-236a-41c4-b0ef-54aff1ab9b8b" />

3. Name the filter disc and select Draw distance. This will present you with a crosshair. Select anywhere on the map of the United States. As you move the crosshair away, you can see a circular area with a radius equal to the movement being selected in the map. Select another point again to complete your geographical criteria.

<img width="431" height="325" alt="image" src="https://github.com/user-attachments/assets/94ebe79f-2d98-43e0-a43e-6bf4b5f75027" />

<img width="585" height="284" alt="image" src="https://github.com/user-attachments/assets/251b0ff8-20bf-4cd4-8882-bd960a5feaba" />

🔍 Why this matters

Geographic filtering is invaluable when your data includes location information. You might use it to analyze traffic from a specific region, investigate incidents near a particular location, or compare behavior across different geographic areas. The filter you create applies to all visualizations on the dashboard, not just the map.

4. You have filtered the web logs data to show only users inside the circle you selected. This filter applies to the map and all other visualizations on the dashboard. This map was created using the Maps visualization editor.

<img width="580" height="753" alt="image" src="https://github.com/user-attachments/assets/7dc39365-450f-4b98-8a07-471bed08fb3a" />

<img width="1960" height="777" alt="image" src="https://github.com/user-attachments/assets/3f673bd9-2e95-4328-afa5-e3603e4bc075" />


🔍 Why this matters

Maps excels at visualizing data with geographic coordinates, supporting multiple layers, custom styling, and various geo-filtering shapes. When your data includes IP addresses (which can be geolocated) or explicit coordinates, Maps helps you discover geographic patterns that might be invisible in traditional charts.
