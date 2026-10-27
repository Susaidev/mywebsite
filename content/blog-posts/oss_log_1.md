+++
title = "OSS contribution log #1"
description= "A deep dive into what my OSS contribution on a reverse ETL tool and the implementation challenges I faced on the way."
date = 2026-10-27

[taxonomies]
tags = ["Data Engineering", "Message Queue", "Reverse ETL"]
+++

## Intro
Open-source software is a cornerstone of software development. All computer science courses materials and textbooks do not exist without open-source software. The production services that support millions of our lives are very like to be built upon open-source operating systems and software components. The technologists you look up to definitely devour technical knowledge present in the open-source software codebase. LLMs were pre-trained over millions of public curated codebases to generate effective code as well. Above all, open-source code defintely made me less clueless about software engineering.
I always wanted to give something back to some open-source movement as a whole since it shaped me who I am today. 

That is exactly why I have started this series, **OSS contribution log**. Every month, I am planning to document my OSS contributions for learning and for your entertainment. Without further ado, let's get started.

## What is the software you contributed to?
DRT is a reverse ETL CLI tool that supports multiple data warehouse sources and data service destinations. A reverse ETL is just a data transformation and transportation logic to move useful, processed data in data warehouses to downstream software services such as CRM, Redis, REST APIs, and Message queues. There are already popular options such as High Touch and Census, but both are closed-source. This tool serves as their open-source alternative and is cli-oriented so modern capable LLM agents can use this tool to perform reverse ETL process easily. 

## My contributions in a nutshell
I added an Amazon Simple Queue Service (SQS) as another data destination. After client developers modifies a YAML file related to SQS configuration, they just need to call a like the following:
```python
event_data = [
{"event_id": "a1", "customer_id": 1, "action": "email"},
{"event_id": "a2", "customer_id": 2, "action": "sell"},
{"event_id": "a3", "customer_id": 3, "action": "purchase"}
]
drt.load(event_data)
```
Then it's done! Client developers can confirm that the load of event data to the selected queue. Here is the detail of [my merged PR](https://github.com/drt-hub/drt/pull/1212).

### Deep dive
Under the hood, there are multiple steps involved:
1. Read a configuration YAML file, and clean event data.
2. Validate the client's input data.
3. Turn the input data into SQS-compatible entry.
4. Send batch of data with retry logic, with different failure modes.
5. Give a response back to the drt's core engine. 

**Record to entry data input conversion:** 
Suppose you want to move processed customer data to CRM. Your data item might look like following:
```json
{"event_id": "a1", "customer_id": 1, "action": "email"},
{"event_id": "a2", "customer_id": 2, "action": "sell"},
{"event_id": "a3", "customer_id": 3, "action": "purchase"},
{"event_id": "a4", "customer_id": 1, "action": "delete_account"}
```

Each dictionary of data is called **record**, which is the expected input format before destination service is used (There might be a drt engine that converts source into this record format).

The client developer also needs to configure a YAML file related to destination service. In this case, a client needs to specify how to group up each record, which queue_url to use, whether queue is FIFO or standard queue, and so on.
With the above data example, the sample YAML might contain the following entries:

```yaml
message_group_id = "customer_id"
deduplication_id_field = "event_id"
```

After parsing YAML file, my coding logic will convert records into **entries**:
```json
{
	"ID": 0, 
	"MessageGroupID": 1, 
	"MessageBody": {"event_id": "a1", "customer_id": 1, ...},
	"MessageDeduplicationId": "a1",
}
...
```

The collection of entries create a **batch**, which is the expected input of [SendMessageBatch API](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html) for SQS. 

#### **Software design decisions and considerations:**
The contribution sounds trivial until you hit multiple constraints. Here are some interesting fixes that I encountered:
1. YAML config data validation 

There are (at least) two different types of queue in SQS: FIFO and standard. Each queue type requires different configurations, so my logic needed to validate the config rigorously before data conversion. For example, FIFO expects MessageGroupID while standard queue does not. 
Define **group** as the entries with the same `MessageGroupID`. FIFO queue ensures that the earlier entry within a group need to be enqueued and dequeued earlier. Since this is unnecessary for standard queue, MessageGroupID should be optional.

2. Different failure modes

In general, there are two different types of failure modes: drt-side failure and SQS failure.
For per-entry failure within a batch, here are multiple possible failures -> fixes:
- A message with more than 10 entries or larger than 10MiB
- Message byte is more than 1MiB or has unserializable data -> Raise a custom error called `RowError`
- Authentication failure -> lazy import and credential chaining.
- Message could not be sent such as due to a client-side network failure. -> use a built-in retry/backoff logic in the codebase.
- Retry is exhausted and a batch cannot be sent -> try sending the next batch.

Another funny thing to point out is the return response of `SendMessageBatch`. The successful response of this method (HTTP code 200) does NOT guarantee that message is sent successfully. Instead, it returns a JSON that contains which data item is successfully sent or not. You can see the [API reference documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SendMessageBatch.html) for more details. Based on this response format, my logic also needed to return the number of successful and failed enqueued items to the sync engine, which eventually showed the result to a client developer. Fortunately, the maintainer of this repo already created counting logic for this so all I did was to call the pre-existing class. 

1. FIFO ordering

Since I skip a batch that cannot be sent with the limited number of retries, there is a case where FIFO nature is violated. The case is when a failed batch contains an earlier entry and a successful batch contains a later entry within the same group. For example:
```json
batch1: {dataA1, dataB1, dataC1, dataA2}
batch2: {dataA3}
```
Suppose both data A1 and data A2 are in the same group and the order must be data A1-> dataA2, there is a possibility that batch 1 cannot be sent with multiple retries and then batch 2 is sent successfully. If data A3 needed to be delivered after data A1 and data A2 for downstream software logic, we would be in trouble. 

To prevent this, I implemented the following:
1. process records in source order
2. **at most** one entry per `MessageGroupId` exists within a batch. 
3. The batch with a permanent failure (e.g. retry exhaustion, server-side failure etc...) updates the `blocked_list` instance attribute, which is a set of `MessageGroupId` whose entry is not sent. If a new batch has an entry whose  `MessaeGroupId` is in `blocked_list`, my logic prevents such batch from being sent and flags `RowError`, and then moves to a next batch to process.

### Wrap-up
That is it! I have never done a proper open-source contribution before so my work might not look impressive. But I am proud of my work, which led to a merged PR. I stayed up late to see the maintainer merge my PR, and once I saw it, I got puuuuumped. 

I have taken another issue from the same repo, so I will definetly keep updating this series once I made decent contribution.
See you next month!