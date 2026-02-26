**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ deployments provide a high availability feature that uses synchronous replication to maintain a standby replica in a different Availability Zone (AZ) from the primary instance. In case of failure, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automatically fails over to the standby replica with minimal downtime. The internal workings involve:

- Automated Failover: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] handles failovers between primary and standby instances without human intervention. During a failover, there is a brief period of read latency increase, followed by connectivity loss, and then automatic recovery.
- Data Replication: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] utilizes database replication technologies like PostgreSQL's warm standby, MySQL's semi-synchronous replication, Oracle's Data Guard, or SQL Server's Database Mirroring to keep data in sync.
- Storage Auto Scaling: Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] automatically scales storage capacity when necessary to ensure application performance.
- Global Databases: For multi-region deployments, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] provides Read Replicas in another region to improve read performance and reduce write latencies.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Characteristics                                                           | When to Use                                                                 |
| --------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ    | Sync replication, auto-failover, within AZ               | High availability for mission-critical applications                         |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas| Asynchronous replication, cross-region, read scalability   | Improve read performance, geographic distribution, decouple reads and writes |
| [[dynamodb]] Global Table   | Multi-region, synchronous replication, automatic failover     | Globally distributed, low latency, highly available applications              |

Anti-pattern: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ as a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] solution instead of using separate regions and implementing backup/restore procedures.

**[[RDS_Instance_Types|3. Security & Governance]]**

Implement cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/dba"
      },
      "Action": [
        "rds-db:connect"
      ],
      "Resource": "arn:aws:rds-db:us-west-2:111122223333:dbuser:database1/*",
      "Condition": {
        "ArnLike": {
          "aws:srcaccount": "123456789012"
        }
      }
    }
  ]
}
```
To enforce organization-wide restrictions, apply Service Control [[policies]] (SCPs):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RDSMultiAzRestrictions",
      "Effect": "Deny",
      "Action": [
          "rds:*"
      ],
      "Resource": [
          "*"
      ],
      "Condition": {
          "StringNotEqualsIfExists": {
              "aws:RequestedRegion": [
                  "us-east-1",
                  "us-west-2"
              ]
          }
      }
    }
  ]
}
```
**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the specific engine used. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies.
For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], combine Multi-AZ deployments with Read Replicas and backups stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved through tagging resources and monitoring usage with AWS [[billing|Cost Explorer]]. Calculate costs using the following formula:

Cost = (Number of DB Instances \* Hours per month \* Instance Type Rate) + (Number of Provisioned IOPS \* P
IOPS rate)

**6. Professional Exam Scenario**

Scenario 1:
You need to create a highly available architecture for a company's mission-critical customer order processing system. The system must have minimal downtime during failures. Which deployment options should you choose?

Correct answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ since it offers automated failover and sync replication within an AZ.
Incorrect answer: Single [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instance because it lacks redundancy and does not offer automated failover.

Scenario 2:
A media company wants to distribute their content across multiple regions while minimizing read latencies. They want to store their video metadata in a relational database. What database deployment option would best fit their needs?

Correct answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas due to its ability to distribute read traffic across multiple regions.
Incorrect answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ alone, as it only provides high availability within a single region.