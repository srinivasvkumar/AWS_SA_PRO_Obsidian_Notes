**Advanced Architecture: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Automated Backups**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups are managed by DB snapshots, which are point-in-time copies of your DB instance. They allow you to recover your database to any time within your retention window. The backup process is internally managed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] and does not impact database performance.

Key architecture components include:

* **Backup Process**: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups create a storage volume snapshot of your DB instance, storing it in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. This backup is stored in the region where your primary DB instance resides.
* **Global Scaling**: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups can be enabled across multiple regions using Multi-Region replication through Read Replicas or through cross-region automated backups.
* **Backup Storage**: Backup storage usage depends on the size of your database and the duration of your [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. It is automatically managed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] and billed separately from your DB instances.

**Comparison & Anti-Patterns**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Backups     | Managed backups for [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] databases                                    |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshots   | Manual, user-initiated backups for [[ec2]] instances                      |
| AWS [[dms]]         | Database migration and synchronization between heterogeneous systems |

Anti-pattern: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] backups as a long-term data archival solution.

**[[appsync|Security]] & Governance**

To implement granular [[appsync|security]] [[policies]], use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with specific permissions. For example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "rds-db:createDBInstanceAutomatedBackup"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access requires setting up an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role that allows the source account to perform actions on the destination account. Use Organizational Service Control [[policies]] (SCPs) to enforce centralized control over AWS resources.

**Performance & Reliability**

When creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups, keep throttling limits in mind. To avoid [[api-gateway|errors]], implement exponential backoff strategies when handling throttled requests.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns include:

* **Multi-AZ deployments**: Provide automatic failover to a standby replica in another AZ.
* **Read Replicas**: Offload read traffic to one or more replicas in different AZs or regions.
* **Backups and Snapshots**: Maintain regular backups and automated snapshots to ensure restore capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include:

* Setting [[billing]] alerts based on custom thresholds.
* Enforcing automated backup retention periods.
* Implementing backup storage tiering.

Example calculation:
Assume 100 GB database, $0.10/GB-month for backup storage, and a 30-day [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. Cost = 100 × 0.10 / 30 = $0.33/month.

**Professional Exam Scenarios**

Scenario 1:
You are designing a highly available e-commerce platform using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]. The company has two separate AWS accounts: Account A contains production services, while Account B hosts [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] infrastructure. How would you configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups to meet these requirements?

Correct answer:
Configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups in Account A and enable cross-account sharing with Account B. Set up a Read Replica in Account B for [[dr]] purposes.

Incorrect answer:
Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] backups in Account B and share them with Account A.

Reasoning:
The first answer follows [[iam|best practices]] by keeping backups close to the primary [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance and implementing [[dr]] measures. In contrast, the second answer places backups further away from the primary [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance, increasing potential latency during recovery.

Scenario 2:
Your organization uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] for various workloads and wants to minimize costs. What steps could you take to optimize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] backup costs?

Correct answer:
Implement backup storage tiering, set [[billing]] alerts, and enforce backup retention periods.

Incorrect answer:
Disable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups altogether to save on backup storage costs.

Reasoning:
The correct answer offers a balanced approach to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] without sacrificing backup functionality. Disabling backups entirely increases risk and violates compliance guidelines.