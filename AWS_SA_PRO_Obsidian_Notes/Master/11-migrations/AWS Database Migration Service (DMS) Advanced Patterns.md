```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[AWS Database Migration Service (DMS)]] Advanced Patterns

#### UNDER-THE-HOOD MECHANICS:
[[AWS DMS]] operates by using replication instances to continuously capture changes from source databases and apply them to target databases. The underlying mechanics include:

- **Change Data Capture ([[CDC]])**: [[dms]] uses CDC to replicate only the changed data, reducing the amount of data that needs to be migrated.
- **Consistency Models**: [[dms]] supports transactional consistency by ensuring that all transactions in a batch are applied atomically at the target database. It also supports point-in-time recovery for transaction logs.
- **Replication Instances**: These are virtual servers running [[DMS agents]], responsible for capturing changes and applying them to the target databases.

#### EXHAUSTIVE FEATURE LIST:
- **CDC**: Continuous replication of only changed data.
- **Full Load Plus CDC**: Initial full load followed by ongoing change replication.
- **Task Settings**: Customize tasks with options like table [[cloudformation|mappings]], filters, transformation rules.
- **Error Handling**: Retry mechanisms and error [[vpc-flow-logs|logging]] for failed migrations.
- **[[appsync|Security]] Features**: SSL encryption for in-transit data, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] integration, [[00_Master_Links_and_Intro|IAM]] roles for access control.
- **Monitoring**: Integration with [[cloudwatch]] for monitoring migration tasks.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set Up Replication Instance:**
   - Launch a replication instance using the AWS Management Console or CLI.
     ```sh
     aws dms create-replication-instance --replication-instance-identifier my-replication-instance --allocated-storage 50 --vpc-security-group-ids sg-1234567890abcdef0 --availability-zone us-west-2a --engine-version "3.4.4" --preferred-maintenance-window sat:03:00-sat:03:30
     ```
2. **Create Source and Target Endpoints:**
   - Configure endpoints for source and target databases.
     ```sh
     aws dms create-endpoint --endpoint-identifier my-source-endpoint --endpoint-type source --engine-name mysql --server-name db-instance-hostname --port 3306 --username admin --password "admin-password" --database-name my-database
     ```
   - Repeat similar steps for the target endpoint.
3. **Define Table [[cloudformation|Mappings]] and Transformations:**
   - Use task settings to map tables from source to target, apply transformations as needed.
4. **Create a Replication Task:**
   - Define and launch replication tasks with specified migration types (Full Load, CDC).
     ```sh
     aws dms create-replication-task --replication-task-identifier my-replication-task --source-endpoint-arn arn-of-source-endpoint --target-endpoint-arn arn-of-target-endpoint --migration-type full-load-and-cdc --table-mappings file://mappings.json
     ```
5. **Monitor and Validate:**
   - Use [[cloudwatch]] metrics to monitor the task progress.
     ```sh
     aws cloudwatch list-metrics --namespace "AWS/DMS" --dimensions Name=ReplicationInstanceIdentifier,Value=my-replication-instance
     ```

#### TECHNICAL LIMITS (2026):
- **Soft Quotas:**
  - Up to 50 replication tasks per replication instance.
  - Up to 30 seconds of lag in CDC for most use cases.
- **Hard Quotas:**
  - Maximum storage capacity per replication instance varies by type (e.g., 1,024 GB for Large instances).
  - 50 [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|security]] groups per region.

#### CLI & API GROUNDING:
- **Create Replication Instance:** `aws dms create-replication-instance`
- **Create Endpoint:** `aws dms create-endpoint`
- **Create Replication Task:** `aws dms create-replication-task`
- **Describe Tasks:** `aws dms describe-replication-tasks`

#### PRICING & TCO:
- **Cost Drivers:**
  - Hourly costs for replication instances.
  - Data transfer costs, especially when moving data across regions.
- **Optimization Strategies:**
  - Use [[00_Master_Links_and_Intro|Spot Instances]] where possible to reduce cost.
  - Optimize storage usage by selecting appropriate instance sizes.

#### COMPETITOR COMPARISON:
- **Azure Database Migration Service ([[dms]])**: Similar capabilities but different pricing models and integration with Azure services. [[dms]] on AWS offers more flexibility in terms of source/target combinations.
- **Google Cloud Data Transfer:** Limited compared to [[AWS DMS]], especially regarding the variety of supported databases.

#### INTEGRATION:
- **[[cloudwatch]] Integration:** For monitoring migration tasks using metrics like replication lag.
- **[[00_Master_Links_and_Intro|IAM]] Integration:** Use [[00_Master_Links_and_Intro|IAM]] roles for secure access control over [[dms]] operations.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Integration:** Securely deploy [[dms]] within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to isolate and protect your environment.

#### USE CASES:
1. **Data Warehouse Migration:** Migrating on-premises data warehouses to [[AWS Redshift]].
2. **Database Consolidation:** Moving multiple databases into a single cloud database for better management.
3. **[[00_Master_Links_and_Intro|Disaster Recovery]] ([[dr]])**: Using [[dms]] to keep replicas up-to-date in case of disaster.

#### DIAGRAMS:
- Placeholder Diagram 1: Architecture Overview
  - [Source DB] -> [[DMS Replication Instance]] -> [Target DB]
- Placeholder Diagram 2: Integration with [[cloudwatch]] and [[00_Master_Links_and_Intro|IAM]]

> [!abstract] Exam Tip
Misunderstanding the differences between Full Load and CDC.
Confusing soft limits (configurable) with hard limits (immutable).
Overlooking the necessity of SSL encryption for secure data transfer.

#### DECISION MATRIX:
| Features                    | Use Case 1: Data Warehouse Migration | Use Case 2: Database Consolidation | Use Case 3: [[00_Master_Links_and_Intro|Disaster Recovery]] |
|-----------------------------|--------------------------------------|------------------------------------|-------------------------------|
| CDC                         | ✓                                    | ✓                                  | ✓                             |
| Full Load                   | ✓                                    | ✓                                  |                              |
| [[appsync|Security]]                    | ✓                                    | ✓                                  | ✓                             |

#### EXAM SCENARIOS:
1. **Scenario:** Migrating a MySQL database to [[Amazon Aurora]].
   - **Question:** How would you configure [[dms]] to ensure minimal downtime?
   - **Answer:** Use Full Load Plus CDC with appropriate table [[cloudformation|mappings]].

2. **Scenario:** Ensuring secure data transfer during migration.
   - **Question:** What features should be enabled?
   - **Answer:** Enable SSL and use [[00_Master_Links_and_Intro|IAM]] roles for [[appsync|security]].

> [!check] Implementation
Use the [[AWS Management Console]] or CLI to configure [[dms]] replication instances, endpoints, tasks, and [[cloudformation|mappings]].

> [!warning] Quota
Soft quotas include up to 50 replication tasks per instance and 30 seconds of lag in CDC. Hard quotas vary by type for storage capacity and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|security]] groups.

> [!danger] Distractor
Real-time event streaming is better suited for [[kinesis|Kinesis Data Streams]] or Kafka, not [[dms]].
For complex ETL workloads, [[00_Master_Links_and_Intro|AWS Glue]] offers more capabilities.
High-frequency data processing should use [[kinesis]] or [[dynamodb]] Streams instead of [[dms]].
Data lake workloads are better handled with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[glue]] ETL pipelines.

#### TIER-3 TROUBLESHOOTING:
1. **[[kms]] Key Policy Conflicts**: Ensure [[00_Master_Links_and_Intro|IAM]] roles have sufficient access to [[kms]] keys.
2. **Replication Task Failures Due to Data Type Mismatch**: Use the AWS Schema Conversion Tool for mapping schema elements.
3. **Network Latency Issues Between Source and Target**: Optimize networking solutions like Direct Connect or [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering.
4. **Resource Exhaustion (e.g., CPU, Memory)**: Monitor resources and scale up replication instances as needed.

#### RECENT UPDATES:
1. **GA Features**:
   - Enhanced support for new database engines such as MariaDB and MongoDB.
   - Improved data type conversions in the AWS Schema Conversion Tool.
   
2. **Deprecated Items**:
   - Certain legacy connectors may be deprecated in favor of newer versions with better performance and [[appsync|security]] features.

By addressing these enhancements, we ensure that the [[nonportable_instructions|review]] is comprehensive, accurate, and relevant for senior-level SAP-C02 exam preparation.