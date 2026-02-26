## Advanced Architecture
At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] is a distributed file system that can be mounted to multiple Linux-based [[ec2]] instances simultaneously. It uses NFSv4 protocol for communication between clients and servers. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] stores data across multiple Availability Zones (AZs) within an region, providing high availability and durability. Each file system has a unique DNS name, which simplifies access control and configuration management.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] provides two performance modes: **General Purpose** and **Max I/O**. General Purpose is suitable for a broad range of workloads, while Max I/O is optimized for high-throughput applications like big data processing, analytics, and media processing. The performance mode can be changed at any time without disrupting application access.

Under the hood, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] leverages a "storage-efficient" variant of NFSv4.1 called "Network File System Version 4.1 over Amazon's Network File System (NFSv4.1 on NFS)" which allows it to deliver consistent performance even as the size of your file system grows.

## Comparison & Anti-Patterns
Here's a comparison table highlighting when not to use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] and potential alternatives:

| Use Case | Don't use [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] if... | Consider using instead... |
|---|---|---|
| Windows Workloads | ...you have a purely Windows environment. | ...Windows Server 2019 Fileshares or [[fsx]] for Windows File Server. |
| High Throughput Applications | ...performance requirements exceed what Max I/O offers. | ...DAX for Redis or [[dynamodb]] for key-value store scenarios. |
| Persistent Block Storage | ...you need block-level storage for your instances. | ...[[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes. |

Common anti-patterns include mounting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] from different regions without proper replication mechanisms in place and assuming [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] is a direct replacement for local instance storage.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] often involve granting cross-account access to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] resources. Here's an example JSON policy snippet allowing another account to manage an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] filesystem:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": [
              "elasticfilesystem:Describe*",
              "elasticfilesystem:CreateMountTarget",
              "elasticfilesystem:DeleteMountTarget"
            ],
            "Resource": "arn:aws:elasticfilesystem:us-west-2:012345678901:file-system/*"
        }
    ]
}
```
For organization-wide governance, Service Control [[policies]] (SCPs) can enforce restrictions on resource creation and modification. For instance, you might prevent anyone from creating unencrypted [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems.

## Performance & Reliability
[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] supports up to 10 GBps throughput and 500,000 IOPS per file system. However, these numbers depend on various factors such as network bandwidth, file size distribution, and concurrent connections. To handle throttling limits, implement exponential backoff strategies in your client code.

High availability can be achieved by distributing mount targets across multiple AZs within a region. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster recovery]] patterns typically involve maintaining standby file systems in separate regions and replicating changes periodically.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] come down to setting appropriate lifecycle management [[policies]]. This includes moving infrequently accessed files to cheaper [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] (e.g., [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] Infrequent Access) and deleting unused snapshots.

Calculation Example: Suppose you store 1 TB of data in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] with 50% cold data. Assuming $0.30/GB-month for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] Standard and $0.10/GB-month for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] Infrequent Access, here's the monthly cost:

- Hot Data: 1 TB * 50% * $0.30/GB-month = $150
- Cold Data: 1 TB * 50% * $0.10/GB-month = $50
Total Monthly Cost: $150 + $50 = $200

## Professional Exam Scenarios
Scenario 1: A media company wants to process large video files in real-time using multiple [[ec2]] instances. They need high throughput and low latency. Which [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] performance mode should they choose? Why?

Correct Answer: Max I/O. This performance mode is optimized for high-throughput applications like media processing.

Incorrect Answers: General Purpose (lower throughput than Max I/O); Provisioned Throughput (not available with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]).

Scenario 2: A pharmaceutical firm needs to securely share sensitive patient records stored in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] across multiple accounts. What steps should they follow?

Correct Answer: Implement cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and bucket [[policies]], ensuring encryption at rest and in transit. Enable versioning for historical record keeping.

Incorrect Answers: Share data via public buckets (insecure); Mount [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] directly between accounts (violates principle of least privilege).