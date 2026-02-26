**AWS Database Migration Service ([[dms]]): Advanced Architecture**

Internally, [[dms]] uses a decoupled architecture with multiple components interacting via AWS services like [[sqs]], [[sns]], and [[kinesis|Kinesis Data Streams]]. The core replication engine is serverless, enabling horizontal scalability to handle massive migration tasks.

Global scale is achieved through the use of replication instances in combination with [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or Direct Connect. This setup allows migrations between continents while minimizing latency and ensuring data durability during transfer.

[[dms]] supports homogeneous migrations (e.g., Oracle to Oracle) and heterogeneous ones (e.g., Oracle to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Aurora]]) using predefined network interfaces called endpoints. These endpoints abstract underlying database engines from the [[dms]] replication instance.

**Comparison & Anti-Patterns**

| Service            | Use Case                                                              |
| ------------------ | -------------------------------------------------------------------- |
| [[dms]]               | One-time or continuous data migration                                |
| [[glue|AWS Glue]] ETL       | Complex transformations before loading into target DB                   |
| Data Pipeline      | Orchestration of multiple AWS services including [[dms]]                |
| Database Migration Operational Readiness Assessment (MORA) | Pre-migration readiness checks               |

Anti-pattern: Using [[dms]] as an ongoing replication mechanism instead of setting up logical replication at the database level.

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dms:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access can be enabled by creating a role in the source account granting permissions to the destination account's [[dms]] role.

Organization Service Control [[policies]] (SCPs):
To restrict [[dms]] usage across an organization, create an [[SCP]] that denies `dms:CreateEndpoint`, `dms:StartMigrationTask`, and `dms:UpdateMigrationTask`.

**Performance & Reliability**

Throttling limits depend on the type of replication instance used. To avoid throttling exceptions, implement exponential backoff strategies when making API calls to [[dms]].

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include deploying replication instances across different Availability Zones (AZs) within the same region or spreading them across regions. For [[dr]] scenarios, use Blue/Green Deployments or Pilot Light strategies.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls involve selecting the right replication instance type based on task requirements. For example, choose a memory-optimized instance for large data warehouses. Also, leverage replication instance placement groups for lower costs.

Calculation Example:
Assuming a single-threaded full load migration task taking approximately 8 hours with 10 GB of data transferred, the estimated cost would be around $0.42 ($0.054 per DMS-run hour * 8 hours + $0.11 per GB transferred).

**Professional Exam Scenarios**

Scenario 1:
A company wants to migrate their production PostgreSQL database to Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] [[aurora]]. They need to maintain minimal downtime and ensure no data loss during the transition. What solution should they adopt?

Correct Answer: Use [[dms]] along with change data capture (CDC) to keep both databases in sync until the cutover point. After validating all data has been replicated, stop writes to the source database and perform a final sync. Finally, switch over to the new [[aurora]] cluster.

Incorrect Answer 1: Perform a one-time bulk export/import operation using tools provided by PostgreSQL. This approach does not guarantee zero downtime nor prevent data loss during the migration process.

Incorrect Answer 2: Leverage [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] for the migration. While [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] simplifies file-based transfers, it does not support database migrations directly.

Scenario 2:
An e-commerce firm plans to set up cross-region replication between two [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] MySQL instances for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes. How can they achieve this efficiently?

Correct Answer: Utilize [[dms]] in conjunction with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] read replicas. Set up a read replica in the secondary region and then configure [[dms]] to replicate changes from the primary [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance to the secondary read replica.

Incorrect Answer 1: Implement custom scripts for periodical data dumps and restores between regions. This method increases management complexity and lacks automation capabilities.

Incorrect Answer 2: Employ AWS [[Storage Gateway]] to cache data in the secondary region. Although this solution provides local access to the data, it doesn't offer automatic failover or synchronization features required for efficient cross-region replication.