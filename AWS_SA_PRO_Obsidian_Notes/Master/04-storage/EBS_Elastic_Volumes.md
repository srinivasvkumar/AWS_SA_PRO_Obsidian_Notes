## Advanced Architecture
At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Elastic Volumes provides block-storage capabilities for [[ec2]] instances. Understanding its [[RDS_Instance_Types|internals]] requires familiarity with terms like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshot, and EBS-optimized instance. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes are network-attached storage for [[ec2]] instances while [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots are point-in-time backups of these volumes. An EBS-optimized instance ensures dedicated throughput to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, maximizing performance.

Global scale is achieved using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Snapshots in conjunction with Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. This allows users to create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes from snapshots across different regions. However, data transferred between regions will incur costs.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Elastic Volumes uses a credit-based model for IO operations. Each volume type has an associated base IOPS value per GB of provisioned storage. For example, gp2 volumes offer up to 3 IOPS per GB, with a maximum of 10,000 IOPS and 16 TiB of storage. If the demand exceeds this limit, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] uses Credit Mechanism where it can burst beyond the base performance at no extra charge as long as there's sufficient credit balance.

## Comparison & Anti-Patterns

| Service          | Use Cases                                                              |
| ---------------- | --------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Elastic Vol | Ideal for boot and operational data for applications requiring low-lat |
| umb & high-through |                                                                     |
| put IO            |                                                                     |
| Instance Store  | Great for temporary or swap data due to no additional charges         |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]              | Suitable for workloads needing NFSv4 protocol, scalable file system   |
| [[fsx]] for Windows  | Preferred choice when demanding high-performance NTFS             |

Anti-patterns include storing infrequently accessed data on high-performance [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, leading to unnecessary expenses. Also, using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes instead of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for backup purposes may result in higher costs and less flexibility.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve permissions related to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] operations such as creating, deleting, attaching, and detaching volumes. Here's a JSON policy snippet allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to perform these actions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVolume",
                "ec2:DeleteVolume",
                "ec2:AttachVolume",
                "ec2:DetachVolume"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access is possible via Resource Share or assuming roles. With [[organizations]] Service Control [[policies]] (SCPs), administrators can enforce restrictions on specific EBS-related activities across accounts.

## Performance & Reliability
Throttling limits depend on the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume type. For instance, gp2 volumes have a baseline of 3 IOPS per GB and a maximum of 10,000 IOPS. To avoid disruptions during heavy load periods, implementing exponential backoff strategies can help manage throttled requests.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns often involve combining [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], or [[lambda]] services. Multi-AZ deployments, read replicas, and Pilot Light [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] architectures are common approaches.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls include choosing the right volume type based on workload requirements, monitoring usage trends, and utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Volume Freeze to prevent accidental deletion or modification of critical volumes. For example, if you don't need high IOPS or availability, consider using cold HDD (sc1) or magnetic (st1) volume types.

## Professional Exam Scenarios

### Scenario 1
An organization wants to move their application stack into a new region for better latency but doesn't want to recreate their existing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes manually. What's the most efficient way to accomplish this?

Correct Answer: Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots of the current volumes and then use them as source images for the new [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes in the target region.

Incorrect Answer: Directly copy the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes to the new region.

### Scenario 2
A company has several AWS accounts under its Organization structure. They want to restrict any non-admin user from creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes larger than 500 GB. How can they achieve this centrally?

Correct Answer: Implement an [[SCP]] at the organization level that denies [[ec2]]:CreateVolume action unless the size parameter is less than or equal to 500 GB.

Incorrect Answer: Modify [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] within each account separately to limit [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume sizes.