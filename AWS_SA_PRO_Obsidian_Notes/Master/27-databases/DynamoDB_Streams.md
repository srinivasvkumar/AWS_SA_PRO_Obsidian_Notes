### Advanced Architecture

At its core, [[dynamodb]] Streams is a log of table activity that captures table item-level changes in real-time. It achieves this by storing an ordered sequence of edit operations for up to 24 hours. The stream data is represented as JSON records, where each record has a key attribute, a type attribute indicating the event that occurred (INSERT, MODIFY, or REMOVE), and optionally, the old image and new image of the modified item.

Internally, [[dynamodb]] Streams uses a partitioned implementation based on the table's hash and range keys. Each partition can handle between 1,000 and 2,000 PUT, GET, and DELETE requests per second before it requires sharding. To achieve global scale, [[dynamodb]] Streams replicates data across multiple AWS Regions using [[dynamodb|DynamoDB Global Tables]]. This enables low-latency reads and writes while maintaining a single digit millisecond latency at the 99th percentile.

### Comparison & Anti-Patterns

| Service | Use Case |
| --- | --- |
| [[dynamodb]] Streams | Real-time data processing, workflow automation, and data warehousing. |
| Amazon [[kinesis|Kinesis Data Firehose]] | Data aggregation, transformation, and loading into storage services like [[Srinivas_Notes/S3|S3]]. |
| Amazon [[sqs]] | Asynchronous message passing between application components. |

Anti-patterns include using [[dynamodb]] Streams when you only need simple notifications of table updates instead of detailed change histories. In such cases, using Amazon [[dynamodb]] Triggers might be more appropriate.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing cross-account access to [[dynamodb]] Streams. Here's an example JSON policy that grants permission to read from a specific [[dynamodb]] Stream in another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:DescribeStream",
                "dynamodb:GetRecords",
                "dynamodb:GetShardIterator",
                "dynamodb:ListStreams"
            ],
            "Resource": [
                "arn:aws:dynamodb:region:account-id:stream/*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "vpc-xxxxxxxx"
                }
            }
        }
    ]
}
```

Cross-account access can also be managed through [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to restrict actions within member accounts.

### Performance & Reliability

[[dynamodb]] Streams have throttling limits based on Shards, which can handle up to 1,000 records per second. If your application exceeds these limits, implement exponential backoff strategies to ensure reliability. For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], enable [[dynamodb]] Streams on [[dynamodb|DynamoDB Global Tables]] and configure AWS Data Stream Manager to replicate data across regions.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include configuring [[dynamodb]] Stream retention periods (from 24 hours up to 7 days) based on your use case. Calculating costs involves estimating the number of PUT, GET, and DELETE requests, along with the amount of data transferred out of [[dynamodb]] Streams. For example, if you store 1 GB of data in a [[dynamodb]] Stream for 24 hours, the estimated cost would be $0.10/GB * 1 GB = $0.10.

### Professional Exam Scenarios

#### Scenario 1

You are designing a serverless web application using [[lambda|AWS Lambda]] and [[dynamodb]]. Your application receives over 10,000 write requests per second during peak times. How would you optimize performance and minimize costs?

Correct Answer: Implement [[dynamodb]] Streams and use [[lambda|AWS Lambda]] to process events in near real-time. Enable auto-scaling for your [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to manage traffic spikes efficiently. By doing so, you can decouple your application components and maintain high performance without overprovisioning resources.

Incorrect Answer: Store all [[dynamodb]] records in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and use [[AWS_SA_PRO_Obsidian_Notes/Master/07-databases/athena|Amazon Athena]] to query them. This approach results in higher latencies due to slower retrieval times compared to [[dynamodb]] Streams. Additionally, it may lead to unnecessary storage costs since [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] charges for data retrieval.

#### Scenario 2

Your company operates in multiple AWS accounts and needs to share data across applications using [[dynamodb]] Streams. What is the most secure way to allow cross-account access?

Correct Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to perform specific actions on the [[dynamodb]] Stream. Attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy granting those permissions to the role. In the destination account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user with required permissions and assume the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account. Finally, set up trust relationships between the two accounts.

Incorrect Answer: Share the [[dynamodb]] Stream ARN directly with other accounts and let them assume the necessary permissions. This approach lacks granularity and does not provide proper auditing capabilities.