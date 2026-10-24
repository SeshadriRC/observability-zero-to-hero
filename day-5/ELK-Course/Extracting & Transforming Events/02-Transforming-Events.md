**Introduction**

In this lesson...

You will:

- Extend your ingest pipeline with processors that transform field values after extraction.
- Convert strings to appropriate numeric types for aggregations, normalize text for consistent querying, and reformat timestamps for readability.

**Note**

These transformations ensure your data is properly typed and formatted for analysis.

<img width="1105" height="443" alt="image" src="https://github.com/user-attachments/assets/be253397-65ac-4b65-87bf-63eaaf9aaba6" />

Transforming Events - (Custom Logs)

In this lab, you will change your ingest pipeline my_redislogs_pipeline to modify data types and format timestamps.

1. In Kibana, go to Stack Management and select Ingest Pipelines.

2. Edit `my_redislogs_pipeline`

<img width="3170" height="928" alt="image" src="https://github.com/user-attachments/assets/0d2c9880-58fb-400f-a7b0-ce57280b17b5" />

3. Create a new processor at the end of the list that converts `redis.pid` from `string` to `long` data type.

🔍 Why this matters

`grok` extracts all values as strings by default. Converting numeric fields to their proper types (like `long` for process IDs) enables numeric aggregations, range queries, and sorting. Without this conversion, "15" would sort alphabetically after "100".

💡Hint

<img width="2680" height="1900" alt="image" src="https://github.com/user-attachments/assets/ee7cb26e-883a-4e4d-abb0-d08e35922b50" />

<img width="1362" height="963" alt="image" src="https://github.com/user-attachments/assets/ee3535ae-0f16-40cf-a9d9-acd3d2074094" />

4. Add a new processor at the end of the list to lowercase the `redis.role` field.

🔍 Why this matters

Normalizing text to lowercase ensures consistent filtering and aggregations. Without normalization, "M" and "m" would be treated as different values, potentially splitting your data in visualizations and queries.

<img width="1375" height="937" alt="image" src="https://github.com/user-attachments/assets/58ed67c8-260a-4496-8ce4-63b17815f1c0" />

5. Include a new processor at the end of the list that creates a `redis.date` field that converts `log_pod.timestamp` from `ISO8601` to the `dd/MM/yyyy HH:mm:ss` format, so it is easier to spot the date of the event.

🔍 Why this matters

The date processor can both parse timestamps into Elasticsearch's internal format and output human-readable formats. Creating a separate `redis.date` field preserves the original timestamp while providing a more readable version for display.

💡Hint

<img width="2690" height="1906" alt="image" src="https://github.com/user-attachments/assets/3ef30a17-46a0-405d-b67a-5ea71d42d6ec" />

<img width="1328" height="882" alt="image" src="https://github.com/user-attachments/assets/8659a955-92f1-4e65-b3ca-731adeb61180" />

6. Next, use the following documents to test your `my_redislogs_pipeline`.

```bash
{
  "_index": "index",
  "_id": "id",
  "_source": {
            "message": "2025-09-26T08:35:42.175821962Z stdout F 1:M 26 Sep 2025 08:35:42.175 * Background saving terminated with success"
  }
},
{
  "_index": "index",
  "_id": "id",
  "_source": {
        "message": "2025-09-26T08:35:42.080268645Z stdout F 15:C 26 Sep 2025 08:35:42.080 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB"
  }
}
```

You should see an output similar to the one below:

<img width="1171" height="952" alt="image" src="https://github.com/user-attachments/assets/d9e77a5d-1b62-4099-9fc0-d90c553e92ef" />

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
          "redis": {
            "date": "26/09/2025 08:35:42",
            "role": "m",
            "log_level": "*",
            "pid": 1,
            "message": "Background saving terminated with success",
            "timestamp": "26 Sep 2025 08:35:42.175"
          }
        },
        "_ingest": {
          "timestamp": "2025-11-06T11:41:07.943855802Z"
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
          "redis": {
            "date": "26/09/2025 08:35:42",
            "role": "c",
            "log_level": "*",
            "pid": 15,
            "message": "Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB",
            "timestamp": "26 Sep 2025 08:35:42.080"
          }
        },
        "_ingest": {
          "timestamp": "2025-11-06T11:41:07.943886413Z"
        }
      }
    }
  ]
}
```

🔍 Why this matters

Compare the output to the previous challenge: redis.pid is now a number (no quotes around 1 and 15), redis.role is lowercase (m and c instead of M and C), and redis.date shows the human-readable timestamp format.

7. The `my_redislogs_pipeline` is fine for now, so click Save pipeline to save it.


**Solution**

```bash
PUT _ingest/pipeline/my_redislogs_pipeline
        {
            "processors": [
                {

                     "grok": {
                       "field": "message",
                       "patterns": [ "%{TIMESTAMP_ISO8601:log_pod.timestamp} %{WORD:log_pod.output} %{CHAR:log_pod.type} %{POSINT:redis.pid}:%{CHAR:redis.role} %{REDISTIMESTAMP2:redis.timestamp} %{REDISLEVEL:redis.log_level} %{GREEDYDATA:redis.message}" ],
                       "pattern_definitions": {
                         "CHAR": "[a-zA-Z]",
                         "REDISLEVEL": "[.\\-*#]",
                         "REDISTIMESTAMP2": "%{MONTHDAY} %{MONTH} %{YEAR} %{TIME}"
                       },
                       "on_failure": [   {  "drop": {}  }  ]
                     }
                   },
                {
                    "remove": {
                        "field": "message"
                    }
                },
                {
                    "convert": {
                        "field": "redis.pid",
                        "type": "long"
                    }
                },
                {
                    "lowercase": {
                        "field": "redis.role"
                    }
                },
                {
                    "date": {
                        "field": "log_pod.timestamp",
                        "formats": [
                            "ISO8601"
                        ],
                        "target_field": "redis.date",
                        "output_format": "dd/MM/yyyy HH:mm:ss"
                    }
                }
            ]
        }

```

**Conclusion**

In this lesson, you added more processors to transform field values, converted strings to numeric types, normalized text for consistent querying, and reformatted the `timestamp` field.


