With Discover, you can quickly search and filter your data, get information about the structure of the fields, and display your findings in a visualization. You can also customize and save your Discover sessions and place them on a dashboard.

In this challenge you will use Discover to analyze logs and keep investigating why users are unable to checkout in the OTel demo webshop.

1. By default, Discover will show the Logs Explorer.

<img width="2600" height="1408" alt="image" src="https://github.com/user-attachments/assets/45b3f683-d947-4750-b153-c5cda3dad039" />

2. Let's switch to Discover since it focuses on general data exploration and analysis.

<img width="2600" height="1410" alt="image" src="https://github.com/user-attachments/assets/43e477cc-ef07-42b6-af50-84dbbaa0e9da" />

3. Use the search bar to filter the results by `service.name: checkout.`

<img width="2602" height="1412" alt="image" src="https://github.com/user-attachments/assets/8ec0dac1-cc90-40e9-829d-68b8d6b04e8c" />
<img width="1356" height="1037" alt="image" src="https://github.com/user-attachments/assets/758403e0-b5a5-4721-a598-668fe1210dad" />

4. Now you can find only log events related to the checkout service, but it might be hard to spot errors. Adjust the query to filter the results by `service.name: checkout and error`.

<img width="2602" height="1410" alt="image" src="https://github.com/user-attachments/assets/005ab28d-5efe-49a4-a654-eb7fe20674ba" />

5. You should see several documents related to errors happening in the checkout service. Expand one of them to take a closer look.

<img width="2602" height="1410" alt="image" src="https://github.com/user-attachments/assets/72fb139a-6bd9-4655-aa89-6da50d1eebee" />

6. It looks like the checkout service is not able to access another service to finish the payment process.

<img width="1088" height="1410" alt="image" src="https://github.com/user-attachments/assets/18630095-0155-408c-8d61-53c40365e9be" />

7. Scroll down to the What's this message? section and let's use the AI Assistant to see if it helps confirm our findings.

<img width="1078" height="1412" alt="image" src="https://github.com/user-attachments/assets/7fc8656c-3218-48c4-9bfd-65428af81be8" />

8. Click What's this message? and you should see an answer like the one below.

<img width="1345" height="987" alt="image" src="https://github.com/user-attachments/assets/cda399af-05d3-4ee3-a023-fd5a3f381be8" />

<img width="667" height="781" alt="image" src="https://github.com/user-attachments/assets/fa02ce6f-417e-4923-bcd7-53d2f14a52d5" />

<img width="667" height="743" alt="image" src="https://github.com/user-attachments/assets/19f09f47-1f12-404f-963c-b0e4d3407ebb" />


9. Did we find the problem?

Click to see the solution

Yes! Take a look at the RPC Error explanation. The checkout service is probably unable to reach the payment service due to a bad address configuration.

10. The AI Assistant helped us confirm that the root cause is a configuration problem, because this possible cause matches the error we just found. This means the checkout problem users are facing in the frontend of the webshop is caused by the checkout service being unable to process payments, because they are not being sent to the correct address. In the next challenge we will use the AI Assistant to confirm our findings.

