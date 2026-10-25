In this lesson...

You will:

- Configure an ILM policy to implement a hot-warm-delete architecture.
- Set rollover conditions, define phase transitions, and configure the delete phase to automatically remove old data.


**Note**

This pattern is common in production environments where you need fast access to recent data while managing storage costs for older data.

<img width="1633" height="583" alt="image" src="https://github.com/user-attachments/assets/d1cd7077-11aa-4052-9299-d907fc7c7af8" />

**Configure ILM**

1. Open the `Advanced settings` of the Hot phase to see its current settings.

<img width="2338" height="1358" alt="image" src="https://github.com/user-attachments/assets/8848539b-71ca-462a-8e1e-b15fcd27edc1" />

You should see that a rollover is set when the index is 30 days old or any primary shard reaches 50 gigabytes. We are far from these two conditions. That's why you saw only one index in the previous lab.

🔍 Why this matters

The default rollover conditions (30 days or 50 GB) are designed for production workloads with significant data volume. Understanding these thresholds explains why backing indices don't roll over frequently in smaller environments—the conditions simply haven't been met.

2. Edit the hot phase to trigger a rollover after 2 minutes only.

🔍 Why this matters

For this lab, we use an aggressive 2-minute rollover to observe the lifecycle in action quickly. In production, rollover conditions balance index size (for query performance) against the overhead of managing many small indices.

💡Hint

Disable `Use recommended defaults` and change `Maximum age` to 2 minutes.

<img width="2388" height="1436" alt="image" src="https://github.com/user-attachments/assets/84c6150a-e745-4b1b-92e0-86fdea5ff0ca" />

3. Next, enable the warm phase and move data into this phase after 0 days. This will force your indices into the warm phase right after the rollover. Set the number of replicas to zero to save some space.

🔍 Why this matters

The warm phase is for data you still need to query but no longer write to. Reducing replicas to zero saves storage space, though it removes redundancy. In production, you'd balance storage savings against the risk of data loss if a node fails.

💡Hint

First, enable the warm phase and set `Move data into phase when`: to 0 days. Then, enable `Set replicas` and set `Number of replicas` to 0.

<img width="2320" height="742" alt="image" src="https://github.com/user-attachments/assets/67681b24-7b0e-4d70-beec-5290d1c385ea" />

4. Finally, delete the indices that are older than 1 hour from rollover.

⚠️ Delete phase permanently removes data

Once an index enters the delete phase, its data is permanently removed and cannot be recovered (unless you have snapshots). In production, ensure your retention period meets compliance requirements and that important data is backed up before deletion.

💡Hint

At the top of the warm phase click the trash can icon to enable the delete phase. In the delete phase set Move data into phase when: to 1 hour.

5. Save the policy.

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/cd306ed9-9335-4d53-9733-1c69c0ccf611" />

<img width="1917" height="1030" alt="image" src="https://github.com/user-attachments/assets/8dbe913e-8e80-4970-b51a-7b7d33aeca0a" />


6. From the Kibana main menu access `Dev Tools` and use `Console` to run the following command, which sets the poll interval for lifecycle policies to 30 seconds:

<img width="327" height="422" alt="image" src="https://github.com/user-attachments/assets/599fa716-c027-49db-8f72-41961dc93ea0" />

<img width="1242" height="271" alt="image" src="https://github.com/user-attachments/assets/d5ca811f-837f-465b-a0ef-fdc2d79c5ef6" />

<img width="1907" height="836" alt="image" src="https://github.com/user-attachments/assets/8a1e44ae-c3b7-430f-8930-79a0e8c98330" />

**Warning**

This is far too low for production. The default of 10 minutes is generally acceptable.

🔍 Why this matters

The poll interval controls how often Elasticsearch checks whether indices should transition phases. A 30-second interval lets you observe lifecycle transitions quickly in this lab. In production, frequent polling adds unnecessary overhead—the default 10 minutes is sufficient since lifecycle transitions aren't time-critical.

**Conclusion**

In this lesson, you configured a custom ILM policy including rollover conditions and phase transitions.
