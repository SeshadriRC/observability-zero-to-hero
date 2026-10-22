**Introduction**

In this lesson, you'll investigate logs from the OTel Demo application components. You'll use Discover to correlate log entries, trace user activity through the application, and understand how different components interact. These skills are essential for debugging distributed applications in production.

**Key Terms**

<img width="1126" height="347" alt="image" src="https://github.com/user-attachments/assets/4bfe3973-34d8-4b84-95ae-53d8bc7ca0f4" />


**OTel Demo Logs**

Use Discover to explore the logs collected by the two integrations on OTel Demo components (Image Provider and Valkey Cart).

1. Navigate to the `OTel Demotab` and place some orders. One of them should have the following item: The Comet Book.

<img width="993" height="500" alt="image" src="https://github.com/user-attachments/assets/9d1de79a-0f65-4e39-9868-394482bf9cf9" />

<img width="1896" height="1043" alt="image" src="https://github.com/user-attachments/assets/339366a3-3b82-425e-a714-352daa570316" />

Generating specific, identifiable activity creates log entries you can search for later. Using a distinctive item name like "The Comet Book" makes it easier to trace the request through the system and verify your queries are working correctly.

2. Navigate back to the Kibanatab and add a filter in Discover: `event.dataset: is one of nginx.access, fs.custom.logs.valkey`

<img width="1437" height="972" alt="image" src="https://github.com/user-attachments/assets/867d8116-1833-413e-b263-786cc865a6bc" />

- Nginx `access.log` entries correspond to OTel Demo frontend visit as Nginx is used to serve the images for OTel Demo shop items.
- `fs.custom.logs.valkey` entries correspond to OTel Demo backend cache

3. Customize the Discover view by adding the following columns: `message`, `url.original`, and `http.request.referrer`

<img width="2618" height="2196" alt="image" src="https://github.com/user-attachments/assets/d0ec2147-2aec-4b99-b2d2-4861ada7d31f" />

<img width="1481" height="1060" alt="image" src="https://github.com/user-attachments/assets/5e15de72-5514-48e0-9153-de0347148038" />

🔍 Why this matters

Customizing Discover's columns lets you focus on the fields most relevant to your investigation. For log analysis, showing fields like event.dataset, message, and relevant URL fields helps you quickly scan results without expanding each document.

4. Enter the query `url.original : /products/TheCometBook.jpg` in the search bar and verify that the http request was initiated by the `Checkout` component.

<img width="2628" height="1616" alt="image" src="https://github.com/user-attachments/assets/11eef242-f579-4ee6-af3f-8831b7ba7152" />

Why this matters

Tracing a specific request through log data demonstrates how you can correlate user actions with backend activity. The http.request.referrer field reveals which page initiated the request, helping you understand the user journey through the application.

<img width="1401" height="1062" alt="image" src="https://github.com/user-attachments/assets/eba53046-0152-40f5-9b7e-ee0cc997b0c9" />

Conclusion

In this lesson, you analyzed OTel Demo logs in Discover to correlate user actions across services and trace application behavior using key log fields.
