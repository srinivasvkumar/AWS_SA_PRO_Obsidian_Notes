**[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] Advanced Architecture**

[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] is a fully managed service that simplifies copying large amounts of data from on-premises storage systems, Amazon Elastic Block Store ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]), Amazon [[fsx]] for Windows File Server, Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] to other supported endpoints. It uses a purpose-built network protocol and agents to perform these tasks efficiently.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] operates at the file level, transferring files and folders while preserving their metadata, such as timestamps, permissions, and ownership. For high performance, it leverages parallel processing and checksums to ensure data integrity during transfers.

[[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] supports multiple global scale scenarios through its hierarchical architecture. At the top level, there's a [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] task that defines the source and destination locations along with any required options. Underneath, a location represents an endpoint in the source or destination environment, which can be an NFS server, SMB server, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]], [[fsx]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket, etc. To facilitate transfers between locations, [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] creates job tasks that execute the actual data movement operations.

**Comparison & Anti-Patterns**

| Service              | Use Case                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------- |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/11-migrations/datasync|AWS DataSync]]         | Large-scale data migration, replication, and synchronization between various storage types |
| AWS Data Transfer    | Migrating large datasets into/out of AWS using physical storage appliances                |
| AWS [[Storage Gateway]] | On-premises cache for cloud storage, backup, and [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]                       |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] when lower-level APIs like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] sync would suffice, or when real-time replication is needed but not provided by [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]].

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing cross-account access for [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]]. Here's a JSON snippet demonstrating how to grant permissions to another account's [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|DataSync components]]:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "datasync:CreateLocationTask",
        "datasync:DescribeLocationTask",
        "datasync:ListTagsForResource"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVault": "arn:aws:s3:::other-account-bucket"
        }
      }
    }
  ]
}
```

Cross-account access can also be restricted using Organization Service Control [[policies]] (SCPs) in combination with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

**Performance & Reliability**

Throttling limits depend on the type of [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] component used. The number of concurrent [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] tasks per account varies based on region. Exponential backoff strategies should be implemented when encountering [[api-gateway|errors]] due to throttling or temporary issues.

HA/DR patterns involve creating multiple [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] tasks to different regions or availability zones, ensuring redundancy and failover capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] usage through [[cloudwatch]] metrics and setting up [[billing]] alarms. Costs can be calculated using the formula: `(Number_of_tasks * Task_duration_in_hours * Number_of_files_or_bytes_transferred) * Cost_per_task_hour`.

**Professional Exam Scenario 1**

Scenario: A company wants to migrate 100 TB of data from their on-premises NAS device to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] using [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]]. They want to optimize costs and minimize downtime. What solution meets these requirements?

Correct Answer: Create a [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] task with the NAS device as the source and an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket as the destination. Schedule the task during non-business hours to minimize downtime. Enable block-level transfers to optimize costs by only sending changed portions of files instead of entire files.

Incorrect Answer: Use AWS [[Storage Gateway]] because it is more suitable for backup and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes rather than large-scale migrations.

**Professional Exam Scenario 2**

Scenario: A customer needs to securely transfer data between two AWS accounts within the same region. How would you set up this configuration using [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]]?

Correct Answer: Set up a [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] task with the source location in one account and the destination location in another account. Grant necessary cross-account permissions using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Optionally, restrict access further using Organization SCPs.

Incorrect Answer: Share the source and destination buckets directly between the two accounts since [[AWS_SA_PRO_Obsidian_Notes/Master/Migration_and_Transfer/DataSync|DataSync]] does not support sharing resources across accounts without proper configurations.