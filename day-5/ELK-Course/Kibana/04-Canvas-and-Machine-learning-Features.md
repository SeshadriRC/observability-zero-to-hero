**Introduction**

In this lesson, you'll explore two additional Kibana capabilities: Canvas for creating presentation-style displays and Machine Learning for detecting anomalies in your data. These features extend Kibana beyond basic visualization into polished reporting and automated insight discovery.

<img width="1362" height="457" alt="image" src="https://github.com/user-attachments/assets/456f3437-4e18-42ea-9194-47952f100077" />

**Canvas and Machine Learning Features**

1. Let's take a look at some additional objects that the Kibana team has created for the web logs dataset. From the Kibana menu, open the Canvas editor by selecting the hamburger menu ( ☰ ) in the top left corner. Under Analytics and select Canvas.

<img width="316" height="608" alt="image" src="https://github.com/user-attachments/assets/78448824-b506-4d7c-99ca-416c592c0f85" />

<img width="1917" height="772" alt="image" src="https://github.com/user-attachments/assets/acf58e29-cdcd-480f-b8d5-b6feb2c6b94a" />

2. Select the workpad titled Logs Web Traffic.

To view this workpad in fullscreen mode, select the fullscreen button in the top right corner of the workpad or View and then Enter fullscreen mode in the center. Alternatively, you can utilize ALT+F.

<img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/8fe9a5e9-a30f-459a-8f61-f9082f1c282e" />

<img width="1917" height="962" alt="image" src="https://github.com/user-attachments/assets/6e4d33fd-1223-46dd-84e5-8458a2d9109f" />

Notice that this is very different from Dashboard in terms of utility. The visualizations are not meant to be interactive. Canvas is great for creating pixel perfect presentations of your live data. However, Canvas is not meant to be used for deep exploration of the data.

🔍 Why this matters

Canvas and Dashboard serve different purposes. Dashboards are for interactive exploration by clicking, filtering, and drilling down into data. Canvas is for presentation by creating visually polished displays for monitors, reports, or executive presentations.

3. Escape the fullscreen mode by using the "esc" key on your keyboard. Next, let's explore how to create a machine learning job. Go back to the home page by selecting the hamburger menu (☰) and then Home

<img width="782" height="226" alt="image" src="https://github.com/user-attachments/assets/596af908-0d1d-4186-bd99-a3ddb3041035" />

<img width="1832" height="445" alt="image" src="https://github.com/user-attachments/assets/b8ec713e-c534-4c90-a1ed-23f5643a77d6" />

Expand the Other sample data sets menu at the bottom of the page. From the Sample web logs data set, click View data then ML jobs.

<img width="751" height="440" alt="image" src="https://github.com/user-attachments/assets/46b4529f-54b1-4162-8de4-bf7dc0d9761e" />

Kibana ships with many pre-built machine learning jobs for known data. This is true for the sample data sets as well. Kibana has already recognized the job that is appropriate for our sample log data and has displayed it here in this wizard. All you have to do is start this job.

Enter the value `logs-prebuilt-` for job ID prefix so that you can identify these jobs easily later and select Create jobs.

<img width="1912" height="673" alt="image" src="https://github.com/user-attachments/assets/c069c558-2406-4cb1-b539-f12e5a19a221" />

🔍 Why this matters

Pre-built ML jobs are designed by Elastic to detect common anomalies in specific data types. For web logs, the job looks for unusual patterns in visitor counts, response times, or geographic distributions. Using a job ID prefix makes it easy to identify and manage related jobs, especially when you have many jobs running across different datasets.

5. Click View results.

<img width="1917" height="712" alt="image" src="https://github.com/user-attachments/assets/1d67fa46-94ca-476f-9721-00183fa45d8e" />

The Anomaly Explorer displays a few of the anomalies that your ML jobs have detected.

🔍 Why this matters

Machine Learning in Elastic automatically establishes baselines for your data and alerts you when behavior deviates significantly. This is far more effective than setting static thresholds, which can miss subtle anomalies or generate false alarms. The Anomaly Explorer helps you investigate what changed and when.

Summary

In this lesson, you installed all of the sample data sets that ships with Kibana. You also saw some of the features of Kibana by exploring the visualizations that loaded with the sample data.
