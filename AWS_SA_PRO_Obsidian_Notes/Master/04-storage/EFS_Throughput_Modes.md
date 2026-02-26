## Advanced Architecture
At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] is a distributed file system that can be mounted to multiple [[ec2]] instances simultaneously. It uses a NFSv4 protocol and stores data in a redundant manner across multiple Availability Zones (AZs) within an region. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] provides a scalable network file system with data synchronously replicated within and across AZs for high availability and durability.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] throughput modes are the two different performance modes available: "General Purpose" and "Max I/O". General Purpose is suitable for a wide variety of use cases and provides a baseline performance of 0.5 MiB/s without any additional charges. Max I/O, on the other hand, provides higher throughput and lower latencies compared to General Purpose but comes at an additional cost.

The throughput mode can be configured when creating a new [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system or by modifying an existing one. The maximum throughput capacity depends on the size of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system, with larger file systems providing higher throughput capacities.

### [[RDS_Instance_Types|Global Scale Considerations]]
[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] offers global capabilities through the use of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] One Zone [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]. This storage class stores data in only one AZ instead of three AZs like the Standard storage class. By doing so, it reduces costs while still offering high durability and availability within a single region. However, One Zone storage class does not provide cross-region replication, which means it cannot offer the same level of data durability as a multi-region setup.

## Comparison & Anti-Patterns
Here are some comparison points between [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] and alternative services such as Amazon [[fsx]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]:

| Service          | Suitable For                           | Data Consistency Model   | Scalability [[Git_hub_notes/certified-aws-solutions-architect-professional-main/12-security-and-config/cloudhsm|Limitations]]     | Cost      |
|------------------|-----------------------------------------|---------------------------|-----------------------------|------------|
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]              | Shared storage for multiple [[ec2]] instances | Eventual consistency       | Highly scalable            | Higher    |
| Amazon [[fsx]]        | High-performance shared storage         | Strong consistency         | Limited scalability          | Lower     |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]              | Single instance block storage             | Strong consistency         | Limited scalability          | Lowest    |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] for single-instance workloads or for workloads requiring strong consistency guarantees. In these cases, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] or Amazon [[fsx]] may be more appropriate.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON policy documents. Here's an example of an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy document for allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to manage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] resources:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "elasticfilesystem:Describe*",
        "elasticfilesystem:CreateFileSystem",
        "elasticfilesystem:DeleteFileSystem"
      ],
      "Resource": "*"
    }
  ]
}
```
Cross-account access can be achieved by using resource-based [[policies]] attached to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system. For example:
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
        "elasticfilesystem:ClientRootAccess"
      ],
      "Condition": {
        "Bool": {
          "elasticfilesystem:AccessedViaMountTarget": "true"
        }
      }
    }
  ]
}
```
Organizational Service Control [[policies]] (SCPs) can be used to enforce restrictions on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] usage across accounts in an [[AWS Organization]]. For example, an [[SCP]] can prevent the creation of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] resources outside a specific set of VPCs.

## Performance & Reliability
Throttling limits depend on the throughput mode and the size of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system. For General Purpose mode, the throughput ranges from 0.5 MiB/s to 10 MiB/s per TiB of data stored. For Max I/O mode, the throughput ranges from 10 MiB/s to 1000 MiB/s per TiB of data stored.

Exponential backoff strategies should be employed when encountering throttling [[api-gateway|errors]]. Backoff intervals should start at a low value (e.g., 1 second) and increase exponentially (e.g., doubling the interval after each failed attempt) up to a maximum limit (e.g., 64 seconds).

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems across multiple AZs and regions. Cross-region replication can be achieved using third-party tools or custom scripts.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be applied by monitoring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] usage metrics and setting alarms based on those metrics. Additionally, using One Zone storage class instead of Standard storage class can reduce costs significantly.

Calculating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] costs involves considering the number of file systems, their size, throughput mode, and data transfer costs. For example, a 100 GiB [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system in General Purpose mode would cost approximately $12.50 per month, excluding data transfer fees.

## Professional Exam Scenarios
Scenario 1:
Your organization has a web application running on [[ec2]] instances in us-east-1. The application requires shared storage accessible from all instances. Due to regulatory requirements, you need to store the data in at least three AZs. Which solution would meet these requirements?

Correct Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] with Standard storage class.

Incorrect Answer: Amazon [[fsx]] because it doesn't support storing data in three AZs.

Scenario 2:
Your company needs to deploy a high-performance computing cluster with shared storage accessible from all nodes. The cluster will generate large amounts of data, and the storage must be highly available and durable. Which storage solution would best fit these requirements?

Correct Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] with Max I/O throughput mode.

Incorrect Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] because it doesn't support shared storage across multiple instances.