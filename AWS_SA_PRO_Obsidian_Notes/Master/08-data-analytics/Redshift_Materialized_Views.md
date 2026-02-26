## [[RDS_Instance_Types|1. Advanced Architecture]]

[[redshift]] Materialized Views are precomputed query results stored as tables in [[redshift]] clusters. They improve query performance by reducing the amount of processing required for analytic queries. The [[RDS_Instance_Types|internals]] involve three primary components:

- **Query optimizer**: Determines if a materialized view should be used based on query complexity, size of underlying data, and staleness of materialized view data.
- **Materialized view engine**: Executes the query, stores the result set, and maintains the view's freshness through incremental updates or time-based snapshots.
- **Metadata catalog**: Tracks metadata about materialized views, such as location, schema, columns, and dependencies.

Global scale can be achieved using Multi-Cluster [[redshift]] with Materialized Views. Each cluster has its own copy of the materialized view, ensuring low latency reads within each region. Data sharing between clusters is possible via Spectrum or Data Exchange.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service                      | When to Use                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| [[redshift]] Materialized Views | For faster querying of large datasets with repetitive queries           |
| [[Collab_Notes_detailed/Analytics/Athena|Athena]] / [[glue]] ETL             | For serverless, ad-hoc querying of data stored in [[Srinivas_Notes/S3|S3]]; or complex transformations |
| [[Timestream]]                   | For [[iot]] sensor data, time series data, or high write throughput workloads  |

### Anti-Patterns

- Using Materialized Views when data changes frequently, causing frequent refreshes and reduced benefits from [[api-gateway|caching]].
- Applying Materialized Views on small datasets, which may not see significant performance improvements.
- Overuse of Materialized Views leading to increased management overhead and storage costs.

## [[RDS_Instance_Types|3. Security & Governance]]

Implement cross-account access to [[redshift]] Materialized Views using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. JSON policy example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "redshift:GetClusterCredentials"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "redshift:DescribeClusters",
                "redshift:DescribeClusterParametersGroups",
                "redshift:DescribeClusterParameterGroups",
                "redshift:DescribeClusterSecurityGroups",
                "redshift:DescribeClusterSubnetGroups",
                "redshift:DescribeClustersDBSecurityGroups",
                "redshift:DescribeOrderableReservedNodeOfferings",
                "redshift:DescribeSnapshotCopies",
                "redshift:DescribeSnapshots",
                "redshift:DescribeTable",
                "redshift:DescribeTables",
                "redshift:DescribeTypeMapAttributes",
                "redshift:DescribeUserDefinedFunctionMetadata",
                "redshift:DescribeUsers",
                "redshift:PauseResumeCluster",
                "redshift:RebootCluster",
                "redshift:ResetClusterParameterGroup",
                "redshift:RestoreFromClusterBackup",
                "redshift:ResizeCluster",
                "redshift:UpdateCluster",
                "redshift:UpdateClusterSoftwareVersion"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Use Service Control [[policies]] (SCPs) at the organization level to restrict access to specific actions, resources, and [[cloudformation|conditions]].

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the node type and number of nodes in your [[redshift]] cluster. Refer to the official documentation for specific limits. Implement exponential backoff strategies during throttled events to ensure reliability.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns include:

- Multi-AZ deployments: Provides automatic failover support.
- Snapshot copies: Allows creating automated and manual backups.
- Cluster replication: Enables real-time backup and restore across different regions.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include:

- Rightsizing [[redshift]] clusters based on workload requirements.
- Monitoring underutilized Materialized Views and removing them when unnecessary.
- Configuring auto-suspend and auto-resume features based on usage patterns.
- Utilizing reserved node pricing for predictable workloads.

Example cost calculations:

- Monthly cost of a single dc1.large node ($0.25 per hour): `0.25 * 24 hours * 30 days = $180`
- Additional data storage cost (assuming 1 TB of data): `$0.25 per GB per month * 1024 GB = $256`

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Vignette 1

Scenario: A retail company wants to perform near-real-time analytics on their sales data while maintaining high availability. They have multiple accounts for production and testing purposes.

Correct answer: Use Multi-AZ deployments for high availability, snapshot copies for [[dr]], and configure periodic snapshot schedules. Set up cross-account access to share Materialized Views between production and test environments. Implement Service Control [[policies]] (SCPs) to enforce proper resource usage.

Incorrect answer: Store all data in one account and rely on manual backup methods, increasing the risk of data loss due to human error.

### Vignette 2

Scenario: A financial company needs to analyze sensitive data stored in [[redshift]] Materialized Views and requires strict [[appsync|security]] measures.

Correct answer: Implement column-level encryption, enable SSL connections, and follow [[iam|best practices]] for securing [[redshift|Amazon Redshift]] clusters. Ensure proper user management and access control [[policies]]. Regularly monitor audit logs for suspicious activities.

Incorrect answer: Ignore [[appsync|security]] concerns and grant public network access to [[redshift]] clusters without proper [[api-gateway|authentication]] or authorization mechanisms.