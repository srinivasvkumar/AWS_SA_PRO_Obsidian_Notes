**Advanced Architecture**

Data Pipeline is a fully managed service that allows you to move and process data between different AWS data stores. It uses activities, which define the actions to be taken on specific data, and schedules, which determine when these activities should occur.

Internally, Data Pipeline makes use of Amazon's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Simple Workflow Service (SWF)]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Simple Notification Service (SNS)]] to orchestrate and notify about task progress. Each activity in a pipeline corresponds to a [[swf]] task, allowing for sequential and parallel execution. Data stored in various AWS services can be accessed using predefined connectors, such as those for Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]], [[dynamodb]], and [[redshift]].

Global scale is achieved by running pipelines within a single region. However, data replication across regions can be implemented manually, allowing for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] and regional failover scenarios. To ensure high availability, multiple instances of each activity can be configured, distributing workload and minimizing single points of failure.

**Comparison & Anti-Patterns**

| Service                     | Use Case                                                              |
| ----------------------------| -------------------------------------------------------------------- |
| [[glue|AWS Glue]]                    | ETL tasks, schema discovery, and [[glue|data catalog]] management            |
| [[aws-batch|AWS Batch]]                   | Running batch computing jobs with custom logic                      |
| [[lambda|AWS Lambda]]                  | Event-driven serverless compute for lightweight transformations   |

Anti-pattern: Using Data Pipeline for real-time streaming data processing. This service is designed for batch processing and scheduled tasks, not for low-latency, stream-based workloads.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created to control fine-grained access to pipelines and underlying resources. For cross-account access, Data Pipeline supports [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles that have permissions to interact with resources in other accounts.

Organization Service Control [[policies]] (SCPs) can be employed to enforce restrictions on creating and updating pipelines at an organizational level.

Example JSON policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "datapipeline:CreatePipeline",
                "datapipeline:DescribePipelines"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**Performance & Reliability**

Throttling limits are enforced per account based on the number of pipelines, tasks, and API calls. If any limit is exceeded, Data Pipeline will return a 429 error code. Exponential backoff strategies should be applied when handling these [[api-gateway|errors]] to ensure reliability.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve configuring multiple instances of each activity, storing data in multiple regions, and implementing monitoring alarms for timely issue detection.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by setting up [[billing]] alerts based on usage metrics, terminating idle pipelines, and optimizing the frequency and schedule of tasks.

Calculation example:
Suppose we run a Data Pipeline instance with one pipeline, 100 tasks per day, and 100 MB of data transfer. The cost would be calculated as follows:

1. Pipeline cost: $10.00/month * (number of days / 30)
2. Task cost: $0.40/task * 100 = $4.00/day * (number of days / 30)
3. Data transfer cost: $0.09/GB * (100 MB / 1024 MB) = $0.000087/day * (number of days / 30)

Total monthly cost: Sum of costs from steps 1-3

**Professional Exam Scenarios**

Scenario 1: A company has two AWS accounts: Account A contains production databases, while Account B hosts development environments. A Data Pipeline job must be set up in Account A to copy data from a production [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] MySQL database to a test [[dynamodb]] table in Account B. How would you implement this?

Correct answer: Create a role in Account B with permissions to write to the test [[dynamodb]] table. Assume that role in Account A's Data Pipeline configuration.

Incorrect answer: Copy data directly from [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] to [[dynamodb]] without assuming a role.

Scenario 2: Your organization has strict governance rules regarding data movement between AWS accounts. How could you enforce these rules using Service Control [[policies]] (SCPs)?

Correct answer: Implement SCPs to restrict Data Pipeline actions, such as `datapipeline:CreatePipeline`, to specific AWS accounts and allow only necessary operations.

Incorrect answer: Attempt to apply SCPs to individual Data Pipeline resources or rely solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].