**Advanced Architecture: [[quicksight]] SPICE**

[[quicksight]]'s Super-fast, Parallel, In-memory Calculation Engine (SPICE) is a scalable, in-memory, columnar storage technology designed for fast querying of data. It decouples compute from storage and auto-scales based on workload demands. Data is automatically organized into an optimized columnar layout, compressed, and stored in memory for low-latency querying. SPICE supports partitioning, filtering, and aggregation operations for efficient data retrieval.

[[RDS_Instance_Types|Global Scale Considerations]]:
- Data is replicated across multiple locations for low-latency access.
- Direct connect or [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] can be used for secure, private access to SPICE datasets.
- [[quicksight]] provides a central place to manage federated single sign-on (SSO) for large [[organizations]].

Under The Hood Mechanics:
- Data ingestion from various sources like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[redshift]], [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]], etc.
- Automatic schema discovery and type inference during dataset creation.
- Data is encrypted at rest and in transit using AES-256 encryption.

**Comparison & Anti-Patterns**

| Service | Use Cases |
|---|---|
| [[quicksight]] SPICE | Interactive dashboards, ad-hoc analysis, machine learning insights. Best when dealing with large datasets (>1GB) that require frequent updates. |
| [[quicksight]] Q | Query-based reporting, ideal for smaller datasets (<1GB) or infrequently updated data. |
| [[kinesis|Kinesis Data Firehose]] + Elasticsearch | Real-time analytics, visualizing streaming data. Not recommended for batch processing due to its real-time nature. |

Common Design Mistakes:
- Using Q instead of SPICE for large datasets.
- Not properly configuring fine-grained access control using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs.
- Ignoring throttling limits leading to unexpected [[api-gateway|errors]] during dashboard usage.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] Example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "quicksight:DescribeDashboard",
                "quicksight:UpdateDashboard",
                "quicksight:CreateDashboard"
            ],
            "Resource": "arn:aws:quicksight:us-east-1:123456789012:dashboard/*",
            "Condition": {
                "StringEquals": {
                    "quicksight:CreatorId": "123456789012"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "quicksight:DescribeDataSet",
                "quicksight:DeleteDataSource",
                "quicksight:CreateDataSource",
                "quicksight:UpdateDataSource",
                "quicksight:CreateDataSourcesEnabled",
                "quicksight:DescribeDataSourcePermissions",
                "quicksight:DeleteDataSourcePermissions",
                "quicksight:DescribeIngestion",
                "quicksight:DescribeDataSetPermissions",
                "quicksight:UpdateDataSetPermissions",
                "quicksight:DescribeResource",
                "quicksight:DescribeGroup",
                "quicksight:DescribeGroupPermissions",
                "quicksight:DescribeIdentityType",
                "quicksight:DescribeNamespace",
                "quicksight:DescribeTemplate",
                "quicksight:DescribeTheme",
                "quicksight:DescribeUserByPrincipalId",
                "quicksight:DescribeUserPermissions",
                "quicksight:ListTagsForResource"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "aws:SourceVpce": "vpce-1234abcd"
                }
            }
        }
    ]
}
```
Cross-Account Access & Organization SCPs:
- Implement cross-account access by creating a role in the source account and allowing required actions.
- Attach [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to the role allowing specific [[quicksight]] actions.
- Create trust relationships between accounts.
- Apply Organization Service Control [[policies]] (SCPs) to restrict access to [[quicksight]] resources.

**Performance & Reliability**

Throttling Limits:
- Maximum number of concurrent sessions per user: 5
- Maximum number of concurrent sessions per dashboard: 10
- Maximum number of concurrent API calls: 500

Exponential Backoff Strategies:
- Implement retry mechanisms using exponential backoff to handle temporary [[api-gateway|errors]].
- Set appropriate timeout values considering the service's response time.

HA/DR Patterns:
- Replicate data across multiple regions for high availability.
- Leverage Amazon [[cloudwatch|CloudWatch alarms]] and event rules to trigger [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:
- Enable usage insights to monitor costs associated with individual users, groups, and datasets.
- Implement resource-level permissions to limit the number of users who can create and share resources.
- Schedule automatic deletion of unused resources using AWS SDK or CLI.

Calculation Examples:
- Assume hourly rate for [[quicksight]] SPICE is $0.0000167 per GB.
- Monthly cost = Daily cost × Number of days in a month
- Daily cost = (Number of active users × Average GB consumed per user) × Hourly rate

**Professional Exam Scenario 1**

Scenario: A company wants to implement a multi-account strategy for their analytics solution using [[quicksight]] SPICE. They want to ensure that only specific users can create and modify resources while maintaining optimal performance.

Correct Answer: Implement cross-account access with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, apply SCPs to restrict access, enable usage insights, and set up automatic deletion of unused resources.

Incorrect Answers:
- Using Q instead of SPICE for large datasets.
- Allowing all users to create and modify resources without proper access restrictions.

**Professional Exam Scenario 2**

Scenario: A gaming company has implemented [[quicksight]] SPICE for real-time analytics but faces unexpected downtime due to high demand during peak hours.

Correct Answer: Implement horizontal scaling using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Auto Scaling]], distribute load using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]], and enable Amazon [[cloudwatch|CloudWatch alarms]] to notify about potential issues.

Incorrect Answers:
- Disabling [[quicksight]] APIs to reduce demand.
- Ignoring throttling limits without implementing exponential backoff strategies.