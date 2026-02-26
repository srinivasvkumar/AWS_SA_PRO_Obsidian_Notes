**Advanced Architecture**

[[kinesis|Kinesis Data Analytics]] (KDA) is a fully managed service that allows you to analyze streaming data using SQL or Apache Flink. KDA has two main components: the application engine and the application blueprint. The application engine runs your code against incoming data, while the application blueprint defines how the application engine should process the data.

Internally, KDA supports both real-time and batch processing modes. Real-time mode processes data continuously as it arrives, while batch mode processes data in batches. KDA integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like [[kinesis|Kinesis Data Firehose]], [[kinesis|Kinesis Data Streams]], and [[cloudwatch]] for monitoring. Global scale can be achieved by deploying applications across multiple regions and leveraging [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]].

Under the hood, KDA uses a pull-based model for data ingestion. It periodically checks for new records in the input stream and pulls them into the application engine. This allows for low latency processing and efficient resource utilization.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[kinesis|Kinesis Data Analytics]] | Real-time analytics of streaming data using SQL or Apache Flink. |
| [[kinesis|Kinesis Data Firehose]] | Data delivery to various destinations without the need for custom code. |
| [[kinesis|Kinesis Data Streams]] | High throughput, durable, and scalable data streams. |
| [[lambda]] | Event-driven serverless compute for non-streaming workloads. |

Anti-patterns include using KDA for non-streaming data, such as querying relational databases, and using it as a general-purpose event processing system instead of more specialized services like [[lambda|AWS Lambda]].

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing cross-account access to KDA resources. Here's an example JSON policy that grants access from another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:root"
            },
            "Action": [
              "kinesisanalytics:DescribeApplication",
              "kinesisanalytics:ListApplications",
              "kinesisanalytics:DiscoverInputSchema"
            ],
            "Resource": "arn:aws:kinesisanalytics:us-east-1:444455556666:application/*"
        }
    ]
}
```

Cross-account access can also be restricted at the organization level using Service Control [[policies]] (SCPs):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "kinesisanalytics:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:SourceVpce": [
                        "org-vpc-endpoint-id"
                    ]
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Throttling limits in KDA depend on the number of application references and the size of the application code. To handle throttled requests, implement exponential backoff strategies with jitter.

HA/DR patterns include deploying KDA applications across multiple regions and using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] for failover. For multi-region deployments, ensure that all dependent resources, such as [[kinesis|Kinesis Data Streams]], are available in each region.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by setting up [[billing]] alarms based on specific usage metrics, such as application runtime hours or input record count. Calculate costs using the following formula:

Cost = Application runtime hours × Number of running applications × Cost per hour per application + Input record processing charges

**Professional Exam Scenarios**

Scenario 1: A company needs to perform real-time analytics on their clickstream data to track user behavior. They have a strict latency requirement of less than 1 second.

Correct answer: Use [[kinesis|Kinesis Data Streams]] as the input source and [[kinesis|Kinesis Data Analytics]] for real-time analytics. Justify: [[kinesis|Kinesis Data Streams]] provides high throughput and low latency, making it suitable for handling clickstream data. KDA can process the data in real-time using SQL or Apache Flink, meeting the latency requirements.

Incorrect answer: Using [[kinesis|Kinesis Data Firehose]] would not meet the latency requirements since it is designed for delivering data to various destinations rather than real-time analytics.

Scenario 2: A media company wants to analyze social media feeds in real-time to detect trending topics during live events. They want to ensure their solution is secure and follows [[iam|best practices]] for governance.

Correct answer: Use [[kinesis|Kinesis Data Streams]] to collect social media feeds and [[kinesis|Kinesis Data Analytics]] for real-time analysis. Implement cross-account access [[policies]] to restrict access to necessary resources. Set up Service Control [[policies]] (SCPs) at the organization level to enforce [[appsync|security]] rules. Justify: [[kinesis|Kinesis Data Streams]] provides high throughput and durability for social media data collection. KDA enables real-time analysis of streaming data using SQL or Apache Flink. Implementing cross-account access [[policies]] and SCPs ensures a secure and governance-compliant solution.