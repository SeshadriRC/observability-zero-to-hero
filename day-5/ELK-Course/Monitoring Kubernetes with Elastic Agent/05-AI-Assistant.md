The AI Assistant is an integration with a Large Language Model (LLM) that helps you understand, analyze, and interact with your Elastic data.

In this challenge you will use the AI Assistant to confirm the checkout problem in the OTel demo webshop.

**Warning**

The suggested prompts can provide different answers due to the nature of the AI Assistant working with live data. Still, we believe you will notice it might be handy to find the root cause of issues. Feel free to adjust the prompts accordingly and help the AI Assistant find the proper direction that helps you.

**Note**

Also note that the conversation is additive. For example, if you ask about a service and then afterwards ask to check transactions, the AI Assistant knows that the two requests are related.

<img width="1282" height="977" alt="image" src="https://github.com/user-attachments/assets/85934f74-e7b7-4895-bb43-d0eacb835396" />

1. Start asking why the ad service is having issues.

```
Why is the "checkout" service having issues?
```

This prompt should provide an answer similar to the one below:


<img width="1654" height="662" alt="image" src="https://github.com/user-attachments/assets/0e97f969-08df-4a59-aba9-6567c40fb26e" />

<img width="1365" height="862" alt="image" src="https://github.com/user-attachments/assets/5fa8cc49-cd19-4f8a-aab8-a60e4a7344f8" />

Since we found something in the logs, let's ask the AI Assistant to take a look there.

2. Ask for a more accurate log analysis.

```
Analyze logs for error messages or unusual patterns.
```

This prompt should provide an answer similar to the one below:

<img width="1672" height="920" alt="image" src="https://github.com/user-attachments/assets/198df510-e4b1-43d5-8e1c-86060868fbde" />

<img width="1372" height="916" alt="image" src="https://github.com/user-attachments/assets/048b56d6-ed91-4e13-8477-11989d158370" />

<img width="668" height="543" alt="image" src="https://github.com/user-attachments/assets/a77837fc-3152-449b-81f1-b22f9c9d38ec" />

The AI Assistant found that RPC error too, which indicates a connection problem between at least two services.


3. Since the checkout service needs to access the payment service to process orders, ask whether it is available.

```
Is payment service available?
```

This prompt should provide an answer similar to the one below:

<img width="1656" height="614" alt="image" src="https://github.com/user-attachments/assets/b7e2e6e3-d083-43b5-99dd-de31aa37548f" />

<img width="1376" height="952" alt="image" src="https://github.com/user-attachments/assets/1f9b0395-7dbc-4aa3-bfa2-90cb527f4189" />

4. Ask about the interaction between these two services.

```
Do you think the checkout service can access payment service?
```

This prompt should provide an answer similar to the one below:

<img width="1668" height="986" alt="image" src="https://github.com/user-attachments/assets/6f462a9b-451b-4c4a-aa54-e442e6ac8925" />

<img width="1406" height="940" alt="image" src="https://github.com/user-attachments/assets/47999861-106f-4b2f-8c17-09506611b0bb" />

Since we know the architecture, we probably found the issue. At some point the checkout service stopped sending payment requests due to the configuration on how to access the payment service. This led to the RPC error we and the AI Assistant found in the logs.

5. Let's tell the AI Assistant our findings and thank for the help provided.

```
Thanks! That's probably the payment service that is unreachable due to misconfiguration in the checkout service.
```

<img width="1666" height="466" alt="image" src="https://github.com/user-attachments/assets/046f35cf-169f-4063-ae1c-979b685ae785" />

The AI Assistant is happy to know it was helpful.

<img width="797" height="417" alt="image" src="https://github.com/user-attachments/assets/b9339918-fb36-4707-802a-c0c51a8632a5" />


6. Congratulations! You reached the end of this training. You learned how to use the Elastic Agent to monitor a Kubernetes environment. You also learned how the AI Assistant can help investigate issues. Feel free to keep exploring the AI Assistant a little bit more.
