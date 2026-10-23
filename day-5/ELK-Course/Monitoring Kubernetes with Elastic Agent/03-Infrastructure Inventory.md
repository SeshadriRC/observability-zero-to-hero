The Infrastructure inventory shows a metrics-driven view of your infrastructure grouped by resource type, while out-of-the-box dashboards are an easy and consolidated way of visualizing node metrics, overall cluster metrics and status across pods.

In this challenge you will explore Kubernetes infrastructure metrics and analyze whether they give us any hints on why the webshop is having issues during checkout.

1. From the Kibana main menu access Infrastructure inventory under the Infrastructure section.

<img width="1312" height="1017" alt="image" src="https://github.com/user-attachments/assets/058f63de-f131-40fa-9183-9ba02432d23b" />

2. The Infrastructure inventory shows Hosts by default. Change it to show Kubernetes Pods instead, as we are interested in checking if any metrics are off with our webshop pods.

<img width="2606" height="1300" alt="image" src="https://github.com/user-attachments/assets/c5a3656b-fe9e-4541-a45b-ca70c2ff1955" />

3. Group by Namespace to make it easier to visualize the pods related to the OTel demo.

<img width="2598" height="1406" alt="image" src="https://github.com/user-attachments/assets/8ec6158b-13d4-49c3-90c2-68a05eb1881a" />

<img width="1917" height="1053" alt="image" src="https://github.com/user-attachments/assets/d70dbb67-3b54-48e3-bfe8-471296e25bac" />

4. The default metric is CPU usage. To make it easier to visualize the pods consuming most CPU, Sort by Metric value.

<img width="2598" height="1402" alt="image" src="https://github.com/user-attachments/assets/28cc75c9-ee75-4446-9776-3c7d00af16a3" />

<img width="1447" height="968" alt="image" src="https://github.com/user-attachments/assets/df72279d-9083-46a3-9d6a-7b85a0ee6e3a" />

5. Now change Metric from CPU usage to Memory usage.

<img width="2598" height="1398" alt="image" src="https://github.com/user-attachments/assets/989f4f18-3602-4289-94f2-be68095488b6" />

<img width="1436" height="1008" alt="image" src="https://github.com/user-attachments/assets/95bb27cc-fc1f-4d46-ac3e-5db05078f0ca" />

6. What are the top 3 OTel demo pods consuming memory? Does that tell us anything?

The top 3 pods with intensive memory usage are: ad, fraud-detection and kafka. Don't worry if your answer is slightly different. All services are expected to be memory intensive. This doesn't give any clues about the checkout issue.

<img width="1436" height="912" alt="image" src="https://github.com/user-attachments/assets/a0d71b3d-5ed9-40d4-b92a-28baed1c369a" />

7. One way to catch Kubernetes pods consuming more memory than they should is by enabling anomaly detection on them. Click Anomaly detection at the top to do that.

<img width="2600" height="1378" alt="image" src="https://github.com/user-attachments/assets/dd134e6d-2cdd-4604-9f60-4c4e39edcfd6" />

8. Next click Enable under Kubernetes Pods to create out-of-the-box machine learning jobs that detect and inspect anomalies in your memory usage and network traffic.

<img width="1338" height="996" alt="image" src="https://github.com/user-attachments/assets/3163584a-ac1e-407c-827b-86decde8fad7" />

9. Click Enable jobs with the default values for creating the ML jobs.

<img width="675" height="706" alt="image" src="https://github.com/user-attachments/assets/78e989a5-e20b-4c07-8481-2d23a6566a35" />

10. After the ML jobs are created, close the panel.

<img width="1442" height="892" alt="image" src="https://github.com/user-attachments/assets/e5ac24d0-5560-4e95-9c7a-f5d695ad5383" />

11. Now let's take a look at the Cluster Overview dashboard since it is another way to monitor the health of the cluster. Either click Kubernetes dashboards at the bottom or access the Dashboards page through the Kibana main menu.

12. Click [Metrics Kubernetes] Cluster Overview to access the dashboard.

<img width="2572" height="1400" alt="image" src="https://github.com/user-attachments/assets/c9e05425-68cf-4271-8883-7a7e87cac840" />

13. In this dashboard you will find useful information about your Kubernetes nodes and namespaces, making it easier to spot problems on specific namespaces by filtering on them, for example.

<img width="1917" height="976" alt="image" src="https://github.com/user-attachments/assets/19b7f6fb-866b-404b-a5ab-d43480c3f22e" />

14. Filter the dashboard by the otel-demo namespace (it may be the default namespace in your environment).

<img width="1908" height="1025" alt="image" src="https://github.com/user-attachments/assets/95ff94b5-da59-4448-9c8f-d14b4a155f67" />

15. Scroll down and check the Top Memory intensive pods.

<img width="2568" height="960" alt="image" src="https://github.com/user-attachments/assets/70b3e034-74e0-4284-bc20-12d4f9188278" />

16. Do you find the same pods from the Infrastructure inventory?

Yes, they should be the same.

17. Even though the metrics haven't provided any insights about the checkout problem, we learned how to check interesting metrics about our Kubernetes cluster. In the next challenge we will keep investigating the problem through Discover.
