**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[sqs]] FIFO queues provide first-in-first-out (FIFO) delivery of messages, guaranteeing message order and ensuring that each message is processed once and only once. Under the hood, FIFO queues consist of partitions, which can handle up to tens of thousands of messages per second. To ensure ordering, messages within a partition are delivered in order, while messages across partitions might not be. Each [[sqs]] FIFO queue has a unique name and requires an additional attribute, `MessageGroupId`, to group related messages together. Optionally, you can specify `MessageDeduplicationId` to prevent duplicate processing of messages.

For global scale applications, you can create identical FIFO queues in multiple regions and replicate messages using Amazon [[eventbridge]] or custom solutions. However, keep in mind that FIFO queues do not natively support cross-region replication.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Cases | Anti-patterns |
|---|---|---|
| [[sqs]] FIFO | Ordered processing, deduplication, synchronous APIs | High throughput, low latency, pub/sub messaging |
| [[kinesis|Kinesis Data Streams]] | Real-time data processing, high throughput, auto-scaling consumers | Ordering, sequential processing |
| [[dynamodb]] Streams | Serverless event-driven workflows, fine-grained events, item-level operations | Large payloads, non-sequential processing |

Common anti-patterns include using FIFO queues for high-throughput systems requiring low latency or pub/sub scenarios. In these cases, consider using [[kinesis|Kinesis Data Streams]] or [[dynamodb]] Streams instead.

**[[RDS_Instance_Types|3. Security & Governance]]**

Ensure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] by restricting actions based on resources and [[cloudformation|conditions]]:
```json
{
    "Effect": "Allow",
    "Action": ["sqs:*"],
    "Resource": ["arn:aws:sqs:*:123456789012:MyFifoQueue"],
    "Condition": {
        "StringEquals": {"aws:SourceVpc": "vpci-abcdefgh"}
    }
}
```
Cross-account access can be granted via resource-based [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "123456789012"
            },
            "Action": "sqs:SendMessage",
            "Resource": "arn:aws:sqs:us-west-2:111122223333:MyFifoQueue",
            "Condition": {
                "ArnLike": {"aws:SourceArn": "arn:aws:iam::123456789012:role/*"}
            }
        }
    ]
}
```
Organization Service Control [[policies]] (SCPs) can enforce restrictions across accounts:
```yaml