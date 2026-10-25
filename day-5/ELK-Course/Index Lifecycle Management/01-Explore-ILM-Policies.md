In this lesson...

You will:

- Explore how Index Lifecycle Management (ILM) policies control the lifecycle of your data.
- Locate the default `logs` policy that manages most log data streams and understand how multiple indices can share the same lifecycle policy.

**Note**

ILM is fundamental to managing storage costs and performance in Elasticsearch deployments.

<img width="1152" height="497" alt="image" src="https://github.com/user-attachments/assets/1d7c3bc9-a276-416d-a735-d8c2efd5642c" />

**Explore ILM Policies**

1. Start by navigating to the ILM page. Under `Stack Management`, click Index Lifecycle Policies.

<img width="1857" height="849" alt="image" src="https://github.com/user-attachments/assets/a55063e1-4761-4ce0-bafc-355c5fe196df" />

<img width="1917" height="832" alt="image" src="https://github.com/user-attachments/assets/39845e8c-57c2-47e1-bc63-b662dc7a46f5" />

🔍 Why this matters

The Index Lifecycle Policies page shows all policies in your cluster, including built-in policies that Elastic provides by default. Understanding what policies exist helps you determine whether to use existing policies or create custom ones for your data.

2. In the previous lab, you found that the `logs-filestream.generic-default` data stream uses the `logs` lifecycle policy. Enable Deprecated and Include managed system policies and search for this policy.

<img width="1857" height="738" alt="image" src="https://github.com/user-attachments/assets/99c36602-4162-4c4d-a97e-6db1e45353a3" />

🔍 Why this matters

The `logs` policy is a built-in policy that Elastic provides for log data. By default, many log integrations use this policy, which means changes to it affect multiple data streams. The filters help you find system-managed policies that are hidden by default.

3. Notice that the `logs-filestream.generic-default` data stream is not the only one using this policy. If you click on the number under Linked indices, you will see all the backing indices using this policy—basically, most of our logs.

<img width="1917" height="846" alt="image" src="https://github.com/user-attachments/assets/9c517566-3ca3-455b-b2de-f0dd3d4da31a" />

Before proceeding to the next step, make sure to return to the Index Lifecycle Policies page by following the above steps.

⚠️ Shared policies affect multiple data streams

Before modifying a shared policy like logs, understand that your changes will affect all linked indices. In production, you might create a custom policy for specific data streams rather than modifying the default policy that many integrations rely on.

4. Click on the three dots and `Edit` to edit the policy.

<img width="1860" height="701" alt="image" src="https://github.com/user-attachments/assets/436a4fcf-db6d-4dfd-ba83-c8e595df82be" />

**Conclusion**

In this lesson, you began to explore how ILM policies control data lifecycle and learned the default ILM policy.
