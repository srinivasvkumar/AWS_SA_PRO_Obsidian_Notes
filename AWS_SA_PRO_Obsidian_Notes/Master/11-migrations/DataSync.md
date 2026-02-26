**Advanced Architecture**

[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] is a fully managed service that simplifies copying large amounts of data across buckets, servers, and accounts. It operates at the file level, supporting various protocols like NFS, SMB, FTP, and HTTP. [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] uses Tasks, Location, and Task Queues for orchestration.

*Task*: Represents the data movement between two locations. It can be scheduled or run on-demand.

*Location*: Represents an endpoint in [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]], which could be an NFS server, an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket, or an FTP/HTTP server.

*Task Queue*: A logical grouping of tasks, allowing you to order task execution sequentially or in parallel.

[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] leverages three types of agents for data transfer:

- *NFS Agent*: Used when the source or destination is an NFS server.
- *[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Agent*: Employed when the source or destination is an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket.
- *Network File System (NFS) iSCSI Discoverable Targets*: Helpful when moving data to AWS [[Storage Gateway]]'s File Share (NFS) target.

Global scale is achieved by having regional endpoints connected via [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]] or public internet. [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] replication jobs maintain object metadata during transfers, ensuring minimal downtime.

**Comparison & Anti-Patterns**

| Service          | Use Case                             |
| ---------------- | ------------------------------------- |
| [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]]         | Large-scale data migration            |
| [[Storage Gateway]] | Hybrid cloud storage                   |
| [[snow|Snowball Edge]]    | Offline, high-latency migrations      |
| [[Srinivas_Notes/S3|S3]] Transfer Accel | Ultra-high-speed transfers           |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] for real-time data replication due to its eventual consistency model.

**[[appsync|Security]] & Governance**

Multi-account strategy involves creating a centralized account with [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] resources while granting required permissions to member accounts. This can be implemented using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and Organizational Service Control [[policies]] (SCPs):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["datasync:CreateLocation*", "datasync:DescribeLocation*"],
      "Resource": "*",
      "Condition": {
        "StringNotEqualsIfExists": {
          "aws:PrincipalArn": [
            "arn:aws:iam::ACCOUNT_ID:role/DataSyncAccessRole"
          ]
        }
      }
    }
  ]
}
```

Cross-account access requires configuring necessary permissions for both location endpoints.

**Performance & Reliability**

Throttling limits depend on the number of active tasks per account and vary based on region. For high availability, distribute tasks among multiple task queues within the same region. Implement exponential backoff strategies during error handling.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] usage through [[cloudwatch]] metrics and setting up alarms for specific thresholds. To calculate costs, multiply the amount of data transferred by the respective regional price.

**Professional Exam Scenarios**

Scenario 1:
You need to migrate 10 TB of data from an on-premises NAS device to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. The source has an NFSv4 share. Which architecture should you choose?

Answer: Use [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] with an NFS agent for the source location and an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket as the destination location.

Scenario 2:
A company wants to move petabytes of data from their existing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket to another account's [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. They want to minimize downtime but also control who can initiate the migration process.

Answer: Create a centralized account with a dedicated [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role for managing [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync]] tasks. Set up an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket location for both source and destination. Grant cross-account access permissions between the two accounts. Finally, create a task queue and schedule the migration task under it.