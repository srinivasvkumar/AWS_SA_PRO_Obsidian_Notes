**Advanced Architecture**

Apache Kafka is a popular event streaming platform for building real-time data pipelines and applications. AWS MSK provides a fully managed service for running Apache Kafka clusters in a familiar and completely compatible way.

Internally, MSK uses Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] to run the Kafka brokers and Zookeeper nodes. Each MSK cluster consists of at least three brokers across three Availability Zones (AZs) within a single Region. Clients connect to one of these broker nodes using an assigned Customer Managed Active Directory (CMAD) ARN or a static list of IP addresses.

Global scale is achieved by deploying multiple MSK clusters across different Regions. To replicate topics between these clusters, you can either build custom solutions using tools like MirrorMaker or utilize third-party services such as Confluent Cloud.

Data durability is ensured through distributed storage and replication. By default, each message is stored on five brokers across two AZs. This configuration ensures that even if an entire AZ goes down, messages remain accessible.

**Comparison & Anti-Patterns**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| MSK             | Real-time data processing, log aggregation, stream processing       |
| [[kinesis|Kinesis Data Firehose]] | ETL, serverless data transformations, integrate with [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] |
| [[eventbridge]]     | Event-driven architecture, schedule tasks, cross-service coordination  |

*Anti-pattern*: Using MSK when simple pub/sub messaging is sufficient. Consider Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Simple Notification Service (SNS)]] instead.

**[[appsync|Security]] & Governance**

*Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]*:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kafka:DescribeCluster",
                "kafka:CreateTopic",
                "kafka:DeleteTopic",
                "kafka:WriteData",
                "kafka:ReadData"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

*Cross-account access*:

1. Create a role in the source account allowing specific actions.
2. Attach this role to the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user or group.
3. Grant permissions to the destination account's users or groups via the assumed role.

*Organization [[appsync|Security]] Control [[policies]] (SCPs)*:

Prevent unauthorized changes to MSK clusters by setting up SCPs at the organization level. For example, deny `kafka:UpdateClusterConfiguration`.

**Performance & Reliability**

*Throttling limits*: Monitor throttled requests under `AWS/MSK` [[cloudwatch]] metrics. Implement exponential backoff strategies when approaching throttle limits.

*HA/DR patterns*:

1. Multi-Region replication using third-party tools like Confluent Cloud.
2. Backup and restore mechanism using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] as a data store.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include configuring the number and size of brokers, retention periods, and monitoring data transfer costs. Calculation examples:

1. Broker instances: Choose the smallest instance type that meets your throughput requirements.
2. [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|Retention period]]: Balance between cost and data availability.
3. Data transfer costs: Enable [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] or private links to avoid public internet charges.

**Professional Exam Scenarios**

Scenario 1:
You need to implement a centralized [[vpc-flow-logs|logging]] solution for a multi-account setup. The solution should be highly available and scalable while minimizing costs. Which architecture would you choose?

Correct answer: Use MSK as a centralized [[vpc-flow-logs|logging]] system integrated with Fluent Bit agents deployed in each account. Apply [[kinesis|Kinesis Data Firehose]] for serverless data transformations and storing logs in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

Incorrect answer: Deploy individual [[kinesis|Kinesis Data Streams]] in each account and send logs directly to the centralized MSK cluster. This approach increases complexity and maintenance efforts due to inter-account communication.

Scenario 2:
Your company operates in multiple geographic regions and requires real-time data replication between MSK clusters. How will you achieve this goal without affecting latency?

Correct answer: Utilize third-party tools like Confluent Cloud or MirrorMaker. These tools provide features to replicate topics across different MSK clusters while maintaining low latencies.

Incorrect answer: Implement custom scripts using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] triggered by [[cloudwatch]] events. This method may introduce higher latencies due to additional processing time.