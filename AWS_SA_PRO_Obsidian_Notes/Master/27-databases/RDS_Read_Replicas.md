**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas utilize a "pull" replication model where the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance acts as the source, and data is asynchronously replicated to one or more Replica instances via MySQL, PostgreSQL, or Oracle engines. The internal architecture involves a Replication Server that runs in the Replica instance, which connects to the source [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance and executes SQL transactions to maintain consistency.

Global [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas enable cross-region replication by creating an additional layer of asynchronous replication over standard [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas. This setup allows read operations to be distributed across multiple regions, improving application latency and regional redundancy.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service                      | Use Case                                                                 |
| ---------------------------- | ------------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas            | Scaling read capacity, offloading read queries from primary DB             |
| [[dms]] / Database Migration     | Migrating data between different databases, heterogeneous workloads       |
| DAX (In-memory cache)        | Low latency read-intensive workloads                                      |
| [[elasticache]] (Redis/Memcached)| In-memory [[api-gateway|caching]] for read-heavy applications                           |

*Anti-pattern*: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas for real-time backups or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]. Instead, consider using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automated backups and DB snapshots.

**[[RDS_Instance_Types|3. Security & Governance]]**

Cross-Account Access:
```json
{
    "Effect": "Allow",
    "Action": [
        "rds-db:connect"
    ],
    "Resource": "arn:aws:rds-db:us-west-2:123456789012:dbuser:database1/*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVpc": "vpc-abcdefghijklmnopq"
        }
    }
}
```
Organization SCPs (to limit [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] creation):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyRDS",
            "Effect": "Deny",
            "Action": [
                "rds:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalOrgID": "<ORG-ID>"
                },
                "Null": {
                    "aws:PrincipalOrgID": true
                }
            }
        }
    ]
}
```
**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:

* Maximum number of read replicas per DB instance: 5
* Maximum number of global read replicas per DB instance: 15 (across all regions)

Exponential Backoff Strategies:

* Implement automatic retries with exponential backoff when encountering throttling [[api-gateway|errors]]

HA/DR Patterns:

* Multi-AZ deployments for automatic failover
* Global [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas for geo-redundancy and low-latency reads

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls:

* Enable performance insights and monitor usage trends
* Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|reserved instances]] and convertible [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|reserved instances]] to optimize costs
* Implement [[billing]] alerts to track monthly spend

Calculation Examples:

* Compute Savings: `(on-demand hourly rate * hours used) - (reserved instance hourly rate * hours used)`
* Storage Costs: `(provisioned storage size * months used * $0.10) + (I/O requests * I/O price)`

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A:
You have a PostgreSQL [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance receiving heavy traffic with 99% write operations. To improve read scalability, you want to set up [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas without affecting the primary database's performance. Which steps should you follow?

Correct Answer:

* Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas in the same region as the source database
* Configure the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas as read-only instances
* Distribute the read load among the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas
* Monitor the replication lag to ensure consistency

Incorrect Answer:

* Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas in a different region as the source database.

Scenario B:
To achieve geo-redundant read-scalable architecture, you plan to create Global [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas in multiple regions. However, due to compliance requirements, you need to restrict [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance creation only within your organization. How can you implement these constraints?

Correct Answer:

* Apply an Organization Service Control Policy ([[SCP]]) denying [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] actions unless the principal ARN matches the ORG ID
* Allow necessary [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] actions within the permitted VPCs

Incorrect Answers:

* Modify the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] attached to individual user accounts
* Implement cross-account access [[policies]] allowing specific IP addresses or CIDR blocks