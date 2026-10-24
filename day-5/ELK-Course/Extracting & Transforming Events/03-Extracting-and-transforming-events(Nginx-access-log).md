In this lesson...

You will:

- Build a complete ingest pipeline for Nginx access logs, applying everything you've learned about grok patterns and transformation processors.
- Parse the standard combined log format, convert response codes and sizes to numeric types, and normalize timestamps. This pipeline demonstrates a realistic workflow for processing web server logs.

<img width="1317" height="392" alt="image" src="https://github.com/user-attachments/assets/662d1fc3-5d5d-47d1-ace8-a9be63daa742" />

## Extracting and transforming events (Nginx access logs)

In this lab, you will learn how to use the grok processor to extract events from Nginx access logs and other processors to modify data types and timestamp formats.

1. First, go to OTel Demo website to buy an item

💡Reminder about OTel Demo architecture and where Nginx (web server) is used

- `Image Provider` is an OTel Demo component using Nginx to serve images to the `Frontend-Proxy` OTel component. Nginx integration collects Logs & Metrics by default, in this challenge, only logs collection is set up.

<img width="3928" height="1686" alt="image" src="https://github.com/user-attachments/assets/3c1d586a-888c-4ee5-9068-d49bf325552a" />

2. Next, use `Discover` to filter logs where `log.file.path` equals to `/var/log/nginx/access.log` and try to figure out the Nginx logs format in the `event.original field`.

🔍 Why this matters

The Nginx integration already parses access logs into structured fields, but the original log line is preserved in `event.original.` Examining this field helps you understand the raw format when building custom pipelines or troubleshooting parsing issues.

💡Hint

You need to search `event.original` in empty fields. This particular field is preserved but not indexed.

<img width="1917" height="987" alt="image" src="https://github.com/user-attachments/assets/626e523e-f7f6-4190-94cf-fc00464e93d6" />

3. You will see web log events that look like the following:

```bash
10.42.0.33 - - [06/Nov/2025:16:20:14 +0000] "GET /products/OpticalTubeAssembly.jpg HTTP/1.1" 200 22201 "https://kubernetes-vm-31047-smqyvkn7awxh.env.play.instruqt.com/cart" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36"
```

Notice that every event has the following format:

```bash
ip_address username user [timestamp] "request" response_code response_size "referer" "user_agent"
```

In that particular example, the first dash after the IP address means that username is not set. The second dash means the user information is also undefined.

🔍 Why this matters

The combined log format is an industry standard used by Nginx, Apache, and other web servers. Understanding this format helps you parse logs from any web server, not just Nginx. The dash characters (-) represent missing or unauthenticated values.

4. To keep things simple, create a pipeline named `my_weblog_pipeline` that parses the `event.original` field with the following `grok` pattern:

```bash
%{IP:ip_address} %{USER:username} %{USER:user} \[%{HTTPDATE:@timestamp}\] "%{DATA:request}" %{NUMBER:response_code} %{NUMBER:response_size} %{GREEDYDATA:everything_else}
```

**Note**

The values in square brackets, like timestamp, will require you to escape a double bracket character (e.g., \[%{HTTPDATE:@timestamp}\]). For now we will not worry about fields after response_size and we will name them everything_else, but we will get back to them in the next labs.

⚠️ Escaping special characters in `grok` patterns

Square brackets have special meaning in regular expressions. To match literal `[` and `]` characters in your log format, you must escape them with backslashes (`\[ and \]`). Forgetting to escape these is a common source of `grok` pattern failures.

💡Hint

First, create a new pipeline and name it `my_weblog_pipeline`.

<img width="2326" height="1244" alt="image" src="https://github.com/user-attachments/assets/c1152776-9dfe-4adc-8ba9-40f675a36469" />


Then, add a `grok` processor that uses the following pattern to extract  `event.original.`


```bash
  %{IP:ip_address} %{USER:username} %{USER:user} \[%{HTTPDATE:@timestamp}\] "%{DATA:request}" %{NUMBER:response_code} %{NUMBER:response_size} %{GREEDYDATA:everything_else}
```

<img width="1420" height="1424" alt="image" src="https://github.com/user-attachments/assets/2207a18b-e507-4bfa-ad60-409ea6104a2f" />

Make sure you click Add to save your `grok` processor.

5. Remove the `event.original` field after it is parsed.

💡Hint

<img width="2360" height="1414" alt="image" src="https://github.com/user-attachments/assets/16158bff-434e-4306-a20b-1bbf71701c33" />

6. Include a new processor at the end of the list that converts `response_code` from `string` to `integer` data type.

🔍 Why this matters

HTTP response codes are numeric values (200, 404, 500, etc.). Converting them to integers enables filtering by ranges (e.g., all 4xx errors) and accurate aggregations in visualizations.

💡Hint

<img width="2368" height="1412" alt="image" src="https://github.com/user-attachments/assets/e5ad911a-9374-4ac2-8938-2545c86112ef" />

7. Add a new processor at the end of the list that converts `response_size` from `string` to `long` data type.

🔍 Why this matters

Response sizes can be large values (megabytes of data), so `long` is the appropriate type. This enables calculating total bandwidth, average response sizes, and identifying unusually large responses.

💡Hint

<img width="2364" height="1412" alt="image" src="https://github.com/user-attachments/assets/7cf3e0c0-48ac-4ffd-bd11-afd6969ee956" />

8. Now, you will test `my_weblog_pipeline` with an existing document. To do that, use the Kibana main menu to open Discover on a new tab and filter logs with the following query:

```
log.file.path : "/var/log/nginx/access.log" and http.response.status_code : 200
```

🔍 Why this matters

Testing with real documents from your index—rather than manually crafted samples—ensures your pipeline handles actual production data correctly, including any edge cases or variations in your real log format.

💡Hint

<img width="2870" height="1512" alt="image" src="https://github.com/user-attachments/assets/dd7b1c0d-601f-4a20-96b0-0004b219cfae" />

9. Copy the `_id` and `_index` of any of these log events.

<img width="2870" height="1530" alt="image" src="https://github.com/user-attachments/assets/96abd349-61b3-4d4e-aaf9-92fe1bb1b150" />

10. Go back to `my_weblog_pipeline`, click `Add documents` and use the `Document ID` from the `Index` that you just copied.

<img width="1082" height="1426" alt="image" src="https://github.com/user-attachments/assets/c56a717a-f4be-4fce-9b2e-b2e7a6af616b" />

11. Test your pipeline and see how your fields are included with the other fields already present in the document. Even though this is not ideal, remember that we are just testing the `grok` processor to see how it works.

12. To make testing `my_weblog_pipeline` easier, edit the remove processor and remove the following fields instead of only removing `event.original:` `agent`, `cloud`, `data_stream`, `ecs`, `elastic_agent`, `event`, `host`, `http`, `input`, `log`, `nginx`, `related`, `source`, `tags`, `url` and `user_agent`.

<img width="2370" height="1418" alt="image" src="https://github.com/user-attachments/assets/c59974f6-3fa6-41d7-8424-137dcfe8eb97" />


⚠️ This removal is for testing only

Removing these fields makes it easier to see your pipeline's output during testing. In a production pipeline, you would typically keep most of these fields—they contain valuable metadata added by the Elastic Agent and integration. Only remove fields you specifically don't need.

13. Refresh the output and see how it is easier now to see the pipeline result without all those extra fields. Remember that we are only doing this for testing purposes.

14. Finally, add a new processor at the end of the list that converts `@timestamp` from the `dd/MMM/yyyy:HH:mm:ss Z` date format to the default `ISO8601` format.

🔍 Why this matters

The `@timestamp` field is Elasticsearch's primary timestamp field used for time-based queries and visualizations. The date processor parses the Nginx timestamp format and converts it to Elasticsearch's expected format, ensuring your logs appear correctly on timelines.

💡Hint

<img width="2364" height="1412" alt="image" src="https://github.com/user-attachments/assets/47a1ce76-7a52-4919-aa91-41bccf3261d5" />

15. Test again your pipeline as you did before or just refresh the output . Note how in the output `response_code` and `response_size` should be numbers now, while `@timestamp` should have the standard `ISO8601` date format.

**Output**

```bash
{
  "docs": [
    {
      "doc": {
        "_index": ".ds-logs-nginx.access-default-2026.10.24-000001",
        "_version": "-3",
        "_id": "6-WobqEBWzY-7f4zgxRB",
        "_source": {
          "request": "POST /api/ui_counters/_report HTTP/1.1",
          "response_code": 200,
          "@timestamp": "2026-10-24T16:04:56.000Z",
          "_tmp": {},
          "everything_else": "\"https://cj87uddfyyry-r5wkyivv43b1.env.play.instruqt.com/app/management/ingest/ingest_pipelines/create\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36\"",
          "ip_address": "10.176.5.147",
          "user": "-",
          "response_size": 15,
          "username": "-"
        },
        "_ingest": {
          "timestamp": "2026-10-24T16:26:59.699748618Z"
        }
      }
    },
    {
      "doc": {
        "_index": ".ds-logs-nginx.access-default-2026.10.24-000001",
        "_version": "-3",
        "_id": "6-WobqEBWzY-7f4zgxRB",
        "_source": {
          "request": "POST /api/ui_counters/_report HTTP/1.1",
          "response_code": 200,
          "@timestamp": "2026-10-24T16:04:56.000Z",
          "_tmp": {},
          "everything_else": "\"https://cj87uddfyyry-r5wkyivv43b1.env.play.instruqt.com/app/management/ingest/ingest_pipelines/create\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36\"",
          "ip_address": "10.176.5.147",
          "user": "-",
          "response_size": 15,
          "username": "-"
        },
        "_ingest": {
          "timestamp": "2026-10-24T16:26:59.699756748Z"
        }
      }
    },
    {
      "doc": {
        "_index": ".ds-logs-nginx.access-default-2026.10.24-000001",
        "_version": "-3",
        "_id": "yOW1bqEBWzY-7f4zSRn6",
        "_source": {
          "request": "POST /_bulk?filter_path=errors%2Citems.%2A.error%2Citems.%2A.status HTTP/1.1",
          "response_code": 200,
          "@timestamp": "2026-10-24T16:18:50.000Z",
          "_tmp": {},
          "everything_else": "\"-\" \"Elastic-metricbeat/8.19.6 (linux; amd64; ac8655385a99413beadaa6ef2447a1203fcb84d8; 2025-10-21 18:56:57 +0000 UTC; Managed)\"",
          "ip_address": "127.0.0.1",
          "user": "-",
          "response_size": 2274,
          "username": "-"
        },
        "_ingest": {
          "timestamp": "2026-10-24T16:26:59.699759323Z"
        }
      }
    }
  ]
}
```
16. The `my_weblog_pipeline` is fine for now, so click Save pipeline to save it.

**Solution (API Version)**

```bash
PUT _ingest/pipeline/my_weblog_pipeline
{
    "processors": [
        {
            "grok": {
                "field": "event.original",
                "patterns": [
                    "%{IP:ip_address} %{USER:username} %{USER:user} \\[%{HTTPDATE:@timestamp}\\] \"%{DATA:request}\" %{NUMBER:response_code} %{NUMBER:response_size} %{GREEDYDATA:everything_else}"
                ]
            }
        },
        {
            "remove": {
                "field": [
                    "event.original",
                    "agent",
                    "cloud",
                    "data_stream",
                    "ecs",
                    "elastic_agent",
                    "event",
                    "host",
                    "http",
                    "input",
                    "log",
                    "nginx",
                    "related",
                    "source",
                    "tags",
                    "url",
                    "user_agent"
                ]
            }
        },
        {
            "convert": {
                "field": "response_code",
                "type": "integer"
            }
        },
        {
            "convert": {
                "field": "response_size",
                "type": "long"
            }
        },
        {
            "date": {
                "field": "@timestamp",
                "formats": [
                    "dd/MMM/yyyy:HH:mm:ss Z"
                ]
            }
        }
    ]
}
```

**Conclusion**

```bash
In this lesson, you created a complete ingest pipeline for Nginx access logs and parsed the standard combined log format, converted response codes and sizes, and normalized timestamps.
```
