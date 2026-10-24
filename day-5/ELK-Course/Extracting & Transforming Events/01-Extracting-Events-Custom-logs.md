Structuring and processing data
Extracting & transforming events

In this lab you will:

- create two ingest pipelines: one for Valkey (Redis) general logs and another for NGINX access logs.
- use the grok processor to extract events from logs.
- transform events with processors ( convert, date, remove, ..)

**Intro**

In this lesson...

You will:

- Create an ingest pipeline that parses Valkey (Redis) general logs using grok patterns.
- Learn to analyze log formats, write patterns that extract structured fields, and handle documents that don't match your expected format.

---

**Note**

Ingest pipelines are the primary mechanism for transforming data as it enters Elasticsearch, making raw logs queryable and analyzable.

<img width="1206" height="451" alt="image" src="https://github.com/user-attachments/assets/832305b2-b716-4783-a932-e6aebb866708" />

---

Extracting Events - (Custom Logs)

In this lab, you will learn how to use the grok processor to extract events from Valkey (Redis) general logs and NGINX access logs. We are using the Custom Logs integration to ingest Valkey (Redis) general logs from /var/log/containers/valkey-*.log, but for now we are just ingesting the events in Elasticsearch without any processing.

1. Start by checking the logs format by accessing the `Logs` app in `Observability` and then filtering logs where `log.file.path` is equal to `/var/log/containers/valkey-*.log`.

🔍 Why this matters

Before writing a `grok` pattern, you need to understand the structure of your log messages. Examining real log samples helps you identify the consistent parts (like `timestamps and log levels`) versus the variable parts (like the actual message content).

💡Reminder about how Valkey (Redis) logs are collected

`Valkey-Cart` component is a cache repository storing related data. The objective is to collect Custom Logs (Filestream) to collect the logs from the Kubernetes pod.

💡Discover Hint

- below image is wrong, so use below command

```bash
log.file.path : "/var/log/containers/valkey-cart-75c49b45c6-m82kr_otel-demo_valkey-cart-680f2874b49ab4228c7e5c20866e57cdb1e5cbc5938281d4f0a854f42a8d97c5.log" 
```

<img width="3732" height="1722" alt="image" src="https://github.com/user-attachments/assets/5abfb363-0e06-4f2e-bf21-9ecb729df2c1" />

- below image i taken from kibana, this is correct

<img width="1475" height="1001" alt="image" src="https://github.com/user-attachments/assets/71dff991-1b2b-4e30-9c6d-1daf07062b18" />

2. There you will see log events that look like these (timestamps will differ):

```bash
2025-09-26T08:35:42.175821962Z stdout F 1:M 26 Sep 2025 08:35:42.175 * Background saving terminated with success
2025-09-26T08:35:42.080268645Z stdout F 15:C 26 Sep 2025 08:35:42.080 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB
2025-09-26T08:35:42.079914612Z stdout F 15:C 26 Sep 2025 08:35:42.079 * DB saved on disk
```
Notice that the pod log files provided under `/var/log/containers` have 2 parts:

- Log part 1 (Pod): 2025-09-25T15:46:17.193900995Z stdout F

- Log part 2 (Redis): 1:M 25 Sep 2025 15:46:17.193 * Background saving terminated with success"

Valkey (Redis) general logs include a timestamp, followed by leading spaces and thread id, the type of the command being executed, and a tab before the actual command.

Such log format can be translated into the following `grok` pattern:

```bash
%{TIMESTAMP_ISO8601:log_pod.timestamp} %{WORD:log_pod.output} %{CHAR:log_pod.type} %{POSINT:redis.pid}:%{CHAR:redis.role} %{REDISTIMESTAMP2:redis.timestamp} %{REDISLEVEL:redis.log_level} %{GREEDYDATA:redis.message}
```

with the following additional `grok` custom definitions:

```bash
{
  "CHAR": "[a-zA-Z]",
  "REDISLEVEL": "[.\\-*#]",
  "REDISTIMESTAMP2": "%{MONTHDAY} %{MONTH} %{YEAR} %{TIME}"
}
```

}

🔍 Why this matters

Grok patterns combine built-in patterns (like TIMESTAMP_ISO8601 and WORD) with custom pattern definitions. When a log format uses non-standard elements—like the single-character Redis role or the custom log level symbols—you define your own patterns to match them.

3. Use the Ingest Pipeline view in Kibana to create a pipeline named `my_redislogs_pipeline` that uses this `grok` pattern to extract Valkey (Redis) general logs from the `message` field.

💡Hints

First, create a new pipeline by going to Stack Management > Ingest Pipelines > Create pipeline > New pipeline and name it `my_redislogs_pipeline`.

<img width="2712" height="1772" alt="image" src="https://github.com/user-attachments/assets/d0a74bf8-584b-4215-bb0e-ea18ef2d6fb4" />

<img width="1472" height="915" alt="image" src="https://github.com/user-attachments/assets/c164108e-7e17-44c6-b3bf-a5e949151034" />

- Then, add a `grok` processor as follows.

<img width="1760" height="2082" alt="image" src="https://github.com/user-attachments/assets/d96d4ac4-088d-48c7-9cbc-9a6970c966ce" />

- Make sure you click Add to save your grok processor.

4. Next, use the following documents to test your `my_redislogs_pipeline`.

```bash
[
{
  "_index": ".ds-logs-fs.custom.logs.valkey-default-2025.09.26-000001",
  "_id": "id1",
  "_source": {
    "message": "2025-09-26T08:35:42.175821962Z stdout F 1:M 26 Sep 2025 08:35:42.175 * Background saving terminated with success"
  }
},
{
  "_index": ".ds-logs-fs.custom.logs.valkey-default-2025.09.26-000001",
  "_id": "id2",
  "_source": {
    "message": "2025-09-26T08:35:42.080268645Z stdout F 15:C 26 Sep 2025 08:35:42.080 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB"
  }
}
]
```

🔍 Why this matters

Testing pipelines with sample documents before deploying them lets you verify your grok patterns work correctly and see exactly what fields will be created. This prevents indexing malformed data into production indices.

💡Hints

First, click `Add documents` to add the testing documents and click `Run the pipeline`.

<img width="1090" height="1266" alt="image" src="https://github.com/user-attachments/assets/41a5f110-2171-43ff-a178-5b2c7fa248ff" />

5. Do all results look good?

💡Hints

You should see an output similar to the one below, which means the results look good:

```bash
  {
    "docs": [
      {
        "doc": {
          "_index": ".ds-logs-fs.custom.logs.valkey-default-2025.09.26-000001",
          "_version": "-3",
          "_id": "id1",
          "_source": {
                "log_pod": {
                            "output": "stdout",
                          "type": "F",
                          "timestamp": "2025-09-26T08:35:42.175821962Z"
                          },
            "message": "2025-09-26T08:35:42.175821962Z stdout F 1:M 26 Sep 2025 08:35:42.175 * Background saving terminated with success",
              "redis": {
                            "log_level": "*",
                            "pid": "1",
      "role": "M",
      "message": "Background saving terminated with success",
      "timestamp": "26 Sep 2025 08:35:42.175"
    }
  },
  "_ingest": {
    "timestamp": "2025-09-26T08:52:36.358154086Z"
  }
  }
},
{
  "doc": {
  "_index": ".ds-logs-fs.custom.logs.valkey-default-2025.09.26-000001",
  "_version": "-3",
  "_id": "id2",
  "_source": {
    "log_pod": {
      "output": "stdout",
      "type": "F",
      "timestamp": "2025-09-26T08:35:42.080268645Z"
    },
    "message": "2025-09-26T08:35:42.080268645Z stdout F 15:C 26 Sep 2025 08:35:42.080 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB",
    "redis": {
      "log_level": "*",
      "pid": "15",
      "role": "C",
      "message": "Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB",
      "timestamp": "26 Sep 2025 08:35:42.080"
    }
  },
  "_ingest": {
    "timestamp": "2025-09-26T08:52:36.358180156Z"
  }
  }
}
]
}

```

<img width="1225" height="955" alt="image" src="https://github.com/user-attachments/assets/8d858385-794e-430f-8f92-aaac8271ec01" />

<img width="1201" height="916" alt="image" src="https://github.com/user-attachments/assets/679ef9cf-8c21-432d-adbc-d9c434df61c5" />

6. The output looks good for now and we can remove the `message` field since it is being parsed correctly. Do that by changing `my_redislogs_pipeline` to remove the `message` field after it is parsed.

🔍 Why this matters

Once you've extracted structured fields from the original message, keeping the raw message wastes storage space. The `remove` processor lets you clean up fields that are no longer needed after parsing.

💡Hints

Click `Add processor`, configure the `remove` processor as follows and click `Add`.

<img width="2872" height="1522" alt="image" src="https://github.com/user-attachments/assets/1a7ea88b-8174-43f5-b35c-3db1ea8e7c41" />

7. Test `my_redislogs_pipeline` one more time, so you can see the `message` field is gone.

💡Hints

First, click View output.

<img width="2816" height="1774" alt="image" src="https://github.com/user-attachments/assets/c6a2837c-5b40-4d4d-914a-afea1b6c0c3f" />

Then, click Refresh output if it is not refreshed automatically.

<img width="1072" height="1448" alt="image" src="https://github.com/user-attachments/assets/be13829e-9f63-4c7e-8d7e-f749518d3f2f" />

8. Sometimes MySQL general logs expand the query body into several lines, which might break our `grok` pattern. To handle this issue in a simple way, add an on failure handler to the on failure `grok` processor for dropping documents that fail to parse.

🔍 Why this matters

Real-world logs often contain edge cases that don't match your expected format—multi-line messages, corrupted entries, or format variations. On-failure handlers let you decide what happens to these documents: drop them, route them elsewhere, or tag them for review.

⚠️ Dropping documents discards data

The `drop` processor permanently discards documents that fail parsing. In production, you might instead use a `set` processor to tag failed documents and route them to a separate index for investigation, preserving the data while keeping your main index clean.

💡Hints

First, click the three dots of the grok processor and select Add on failure handler.

<img width="2852" height="1792" alt="image" src="https://github.com/user-attachments/assets/83063f03-c02c-4364-80a0-b9b279261f66" />

Then, configure the on failure handler with a `drop` processor and click `Add`.

<img width="1430" height="1428" alt="image" src="https://github.com/user-attachments/assets/0c834dae-9733-4e33-be50-452d31113b24" />

<img width="1177" height="876" alt="image" src="https://github.com/user-attachments/assets/02c1b6a9-d159-4a4d-8087-c670eb0d8ce3" />

9. The `my_redislogs_pipeline` is fine for now, so click `Create pipeline` to save it. In the next labs you will be adding more processors to this pipeline, so at the end of this module you will be able to use it in the Custom Logs integration to properly parse Redis (Valkey) general logs.

<img width="1915" height="955" alt="image" src="https://github.com/user-attachments/assets/3e53f45c-e774-4c32-aed5-b16fd8f062ba" />

---

**Solution (API Version)**

```bash
PUT _ingest/pipeline/my_redislogs_pipeline
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{TIMESTAMP_ISO8601:log_pod.timestamp} %{WORD:log_pod.output} %{CHAR:log_pod.type} %{POSINT:redis.pid}:%{CHAR:redis.role} %{REDISTIMESTAMP2:redis.timestamp} %{REDISLEVEL:redis.log_level} %{GREEDYDATA:redis.message}"
        ],
        "pattern_definitions": {
          "CHAR": "[a-zA-Z]",
          "REDISLEVEL": "[.\\-*#]",
          "REDISTIMESTAMP2": "%{MONTHDAY} %{MONTH} %{YEAR} %{TIME}"
        },
        "on_failure": [
          {  "drop": {}  }
        ]
      }
    },
    {
      "remove": {  "field": "message" }
    }
  ]
}
```

**Conclusion**

In this lesson, you created an ingest pipeline to parse Valkey general logs, analyzed log formats, wrote patterns for structured fields, and handled documents that don't match expected format.
