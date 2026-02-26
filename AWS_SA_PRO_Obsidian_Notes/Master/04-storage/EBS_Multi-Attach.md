**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach allows simultaneous access to an Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume from multiple [[ec2]] instances within a single Availability Zone. This is achieved by using the PI (Partition Interleave) protocol that coordinates access across multiple nodes. The PI protocol uses a cluster of journals to capture changes made by each node. Each journal contains its own copy of the file system metadata, allowing independent updates without conflict.

[[RDS_Instance_Types|Global scale considerations]] include the following:

* Data consistency: Changes made by one node become immediately visible to other nodes. However, there might be a delay in propagating these changes due to network latency or PI protocol operations.
* Performance: The maximum throughput achievable depends on the number of nodes attached to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume and their individual performance characteristics.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Cases                                                                                       |
| --------------- | --------------------------------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach | Highly available applications requiring data redundancy, such as databases, NoSQL systems   |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]              | Applications requiring high levels of concurrent access, like web servers, content management |
| [[fsx]] for Windows  | Windows-based applications needing high-performance, enterprise-grade storage            |

Anti-patterns include storing transient data on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes since they don't provide replication across AZs. Also, attaching [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes to instance types not supporting it (e.g., T2, R5) is another anti-pattern.

**[[RDS_Instance_Types|3. Security & Governance]]**

To enable cross-account access, you need to create a resource policy attached to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume, allowing specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles from other accounts to perform certain actions. Here's an example JSON policy:

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
                "ec2:AttachVolume",
                "ec2:DetachVolume"
            ],
            "Resource": "arn:aws:ec2:us-west-2:111122223333:volume/*",
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/Dept": "Finance"
                }
            }
        }
    ]
}
```

Additionally, [[organizations]] can enforce restrictions via Service Control [[policies]] (SCPs):

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:DescribeVolumes",
      "Resource": "*",
      "Condition": {
        "StringNotEqualsIgnoreCase": {
          "ec2:Region": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

The throttling limit for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach is 16 attachments per volume. If you reach this limit, consider splitting your data into smaller segments and distributing them across multiple [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes. To handle throttling exceptions, implement exponential backoff strategies using the AWS SDKs.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], spread [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes across different AZs and ensure proper backup procedures are in place.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be implemented by tagging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes and using AWS [[billing|Cost Explorer]] to analyze spend. Additionally, enabling deletion protection prevents accidental deletions, thus avoiding unnecessary costs.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
A company wants to deploy a highly available MySQL database with read-write splitting across two availability zones. They need to minimize data loss while maintaining high performance. Which solution would you recommend?

Correct Answer: Implement MySQL Master-Slave replication with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volumes serving as shared storage between the master and slave instances. This setup ensures data consistency while providing high availability and performance.

Incorrect Answers: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] for shared storage wouldn't work because it doesn't support multi-attach. Implementing MySQL Multi-AZ deployment does not offer read-write splitting capabilities.

Scenario 2:
An organization has strict compliance requirements prohibiting the storage of sensitive data outside their primary account. How can they securely share an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volume with a separate AWS account while adhering to these constraints?

Correct Answer: Create a resource policy attached to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume granting specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles from the secondary account access to perform attach/detach actions. Implement cross-account [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with permissions limited to the necessary actions.

Incorrect Answers: Sharing the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] Multi-Attach volume directly with the secondary account without a resource policy would expose the resource to unauthorized access. Allowing full [[ec2]]:DescribeVolume access to all resources could lead to unintended information disclosure.