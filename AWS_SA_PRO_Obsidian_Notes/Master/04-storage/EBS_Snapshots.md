**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshots are point-in-time backups of Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. They leverage copy-on-write (CoW) technology, creating block-level deltas between snapshots. This allows efficient storage and quick creation times. However, deleting a snapshot may cause all dependent snapshots to also be deleted due to the hierarchical structure. To avoid this, use "createsSnapshot" API instead of directly modifying snapshots.

Global [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshots can be achieved using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or [[Storage Gateway]], enabling migration across regions while maintaining data integrity. For large-scale operations, consider [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] which supports policy-driven [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshot management across accounts and regions.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Case                                           |
| --------------- | ------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshots   | Fast, incremental backups within same region        |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]]      | Centralized backup management, cross-region        |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]      | Long-term archival, regulatory compliance            |
| Instance Store | Low-latency, high-throughput, transient workloads |

Anti-pattern: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots as primary storage or relying solely on them for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] without considering RPO/RTO requirements.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy Example: Allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to create but not delete [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots:

```json
{
    "Effect": "Allow",
    "Action": ["ec2:CreateSnapshot"],
    "Resource": ["*"]
},
{
    "Effect": "Deny",
    "Action": ["ec2:DeleteSnapshot"],
    "Resource": ["*"]
}
```

Cross-account sharing can be enabled through Resource [[policies]] on the source account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {"AWS": "arn:aws:iam::target-account:root"},
            "Action": ["ec2:DescribeSnapshots"],
            "Condition": {"StringEquals": {"ec2:Owner": "source-account"}},
            "Resource": "arn:aws:ec2:region::snapshot/*"
        }
    ]
}
```

Organization SCPs can enforce tagging [[policies]] and prevent accidental resource exposure.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits vary based on region and instance type. If you encounter throttling issues, implement exponential backoff strategies using SDKs or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

HA/DR patterns involve frequent snapshots, synchronous replication (using Multi-Attach), or integration with services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]].

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost control involves applying lifecycle [[policies]] to automatically transition snapshots to lower-cost [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] post-creation. Calculate costs using the Pricing Calculator or AWS [[billing|Cost Explorer]].

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A: An organization has multiple AWS accounts managing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes. They want to centrally manage their [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots and ensure proper backup [[policies]]. What solution should they adopt?

Correct Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]], allowing centralized management of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots and policy-based automation.

Incorrect Answer: Create manual scripts to copy [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots across accounts, leading to operational complexity and potential inconsistencies.

Scenario B: A company is concerned about accidental deletion of critical [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots by its developers. How can they restrict these actions while still allowing developers to create new snapshots?

Correct Answer: Implement an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that denies the "[[ec2]]:DeleteSnapshot" action for developers.

Incorrect Answer: Disable snapshot functionality entirely for developers, hindering productivity and innovation.