### Advanced Architecture

At its core, [[kinesis|Kinesis Data Streams]] is a distributed, fault-tolerant, and scalable service that can continuously capture and store terabytes of data per day from hundreds of thousands of sources such as application servers, websites, and connected devices. It achieves this by sharding data across multiple partitions based on the key provided during record input. The number of partitions depends on the projected total data volume, and it's crucial to plan this effectively as increasing partition count later requires recreating the stream.

Internally, [[kinesis]] uses an "under the hood" mechanism called "Consumer Coordination" which ensures each record in the stream is processed once by only one consumer. This is achieved using Amazon [[dynamodb]]'s consistent hashing algorithm.

[[RDS_Instance_Types|Global scale considerations]] include Multi-Region replication and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] scenarios. With Multi-Region replication, you can have up-to-date copies of your data in multiple regions. However, keep in mind that writes are more expensive due to replication costs. For [[dr]], you could set up a secondary [[kinesis]] stream in another region and configure your consumers to switch over when needed.

### Comparison & Anti-Patterns

| Service | Use Case |
|---|---|
| [[kinesis|Kinesis Data Streams]] | Real-time processing of large-scale, streaming data ingestion. High throughput, durability, and fault tolerance required. |
| [[kinesis|Kinesis Data Firehose]] | Simplified data loading to destinations like [[Srinivas_Notes/S3|S3]], [[redshift]], Elasticsearch, etc., without needing to manage individual consumers or checkpoints. |
| [[kinesis-video-streams|Kinesis Video Streams]] | Processing video data from [[iot]] devices. |

Anti-pattern: Using [[kinesis|Kinesis Data Streams]] when simple event-driven architecture would suffice with services like [[sns]] or [[sqs]].

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing cross-account access. Here's a JSON snippet demonstrating how to allow a role in Account B to write to a [[kinesis]] Stream in Account A:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::B-Account-ID:role/RoleName"
            },
            "Action": [
                "kinesis:DescribeStream",
                "kinesis:PutRecord",
                "kinesis:PutRecords"
            ],
            "Resource": "arn:aws:kinesis:us-east-1:A-Account-ID:stream/MyStreamName",
            "Condition": {
                "ArnLike": {
                    "aws:SourceVpc": "arn:aws:ec2:us-east-1:B-Account-ID:*"
                }
            }
        }
    ]
}
```

Cross-account access can also be controlled via Organization Service Control [[policies]] (SCPs) denying specific actions.

### Performance & Reliability

Throttling limits depend on the number of shards used by the [[kinesis]] Data Stream. Each shard provides 1MB/sec ingress and 2MB/sec egress. Consumers should implement exponential backoff strategies to handle throttling exceptions.

High availability/disaster recovery patterns typically involve having multiple consumers in different AWS Regions reading from the same [[kinesis]] Data Stream.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls come from understanding usage patterns and scaling down the number of shards during periods of low activity. Also, deleting unused streams reduces costs significantly.

Calculation example: If you have a stream with 24 shards and pay $0.024 per hour for each shard, then the total monthly cost will be around $(24 * $0.024 * 24 * 30) = $172.8 approximately.

### Professional Exam Scenario

Scenario 1: Your company operates globally with several AWS accounts managing various applications. They want to centralize real-time analytics processing but maintain regional data sovereignty. How would you design this?

Correct answer: Create a central account holding a [[kinesis]] Data Stream. Set up cross-account roles allowing each regional account to write to the central stream. Ingest regional data into their respective [[kinesis]] Streams first before sending it to the central stream.

Incorrect answer: Send all regional data directly to the central [[kinesis]] Stream. This approach doesn't respect regional data sovereignty requirements.

Scenario 2: Your system receives millions of events per second. After processing, these events need to be stored in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Which service(s) should you use and why?

Correct answer: Instead of [[kinesis|Kinesis Data Streams]], use [[kinesis|Kinesis Data Firehose]] to send records directly to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. This simplifies architecture, avoids managing individual consumers and checkpoints, and scales automatically based on incoming data volume.

Incorrect answer: Using [[kinesis|Kinesis Data Streams]] followed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] writing to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] adds unnecessary complexity and potential points of failure.