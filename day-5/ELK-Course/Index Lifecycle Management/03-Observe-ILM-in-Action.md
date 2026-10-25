In this lesson...

You will:

- Observe your ILM policy in action by watching indices roll over and transition through phases.
- See how Elasticsearch automatically creates new backing indices and moves older ones from hot to warm.

**Note**

This hands-on observation reinforces how ILM automates index management without manual intervention.

<img width="1616" height="345" alt="image" src="https://github.com/user-attachments/assets/e480209d-3b43-46a6-8189-f840ccd028c8" />

**Observe ILM in Action**

1. Navigate to **Index Management**, and filter the **Indices** to visualize the backing indices of the **logs-filestream-generic-default** data stream. You can use the following query:

```bash
data_stream="logs-filestream.generic-default"
```


**Note**

It may take a few moments for indices to appear. Select Reload indices every minute or so until they appear.

🔍 Why this matters

Filtering by data stream name shows you all the backing indices that store data for that stream. This view lets you monitor rollover behavior and verify that your lifecycle policy is creating new indices as expected.

💡Hint

Paste the filter in the query bar and don't forget to enable Include hidden indices.

<img width="1917" height="801" alt="image" src="https://github.com/user-attachments/assets/becba5aa-f7a4-47ba-bead-20e5fffc7f21" />

2. Wait a bit before hitting the Reload indices button. After a few minutes, you should see that a new backing index has been automatically created.

🔍 Why this matters

The new backing index appears because your rollover condition (2 minutes) was met. Elasticsearch automatically created a new index and redirected writes to it. This happens without any manual intervention—ILM handles it based on your policy.

<img width="1916" height="855" alt="image" src="https://github.com/user-attachments/assets/79d70ef8-5276-4546-90ac-b31d12c204e7" />

3. Open the latest index, and you should see that its current phase is Hot.

<img width="1917" height="797" alt="image" src="https://github.com/user-attachments/assets/3a624e33-4b6d-4c11-89e7-ec4df004f43d" />

🔍 Why this matters

The latest (current) backing index is always in the hot phase because it's actively receiving writes. Only this index accepts new documents for the data stream; older backing indices are read-only.

4. If you open the other indices that are not the current one, you should see that they are in the Warm phase. After 1 hour, you will eventually see those indices disappear.

🔍 Why this matters

Older indices transition to warm immediately after rollover (as configured with 0 days). They remain queryable but no longer receive writes. When they reach 1 hour old, the delete phase removes them entirely, freeing storage space automatically.

<img width="1917" height="856" alt="image" src="https://github.com/user-attachments/assets/0a36f21a-f84e-46f1-ac7b-00e4eff00b09" />
