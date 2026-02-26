### Advanced Architecture

At its core, Amazon [[fsx]] for Windows File Server is a fully managed native Microsoft Windows file server service, providing a shared file system accessible via the industry-standard Server Message Block (SMB) protocol. It supports Windows-based applications and workloads that require file storage, offering feature compatibility with Windows File Server, including full support for Active Directory (AD) integration, Windows Access Control Lists (ACLs), and Distributed File System (DFS).

[[fsx]] for Windows File Server uses two types of storage options: SSD and HDD. The SSD option provides high IOPS and low latency suitable for performance-critical applications such as databases or content creation workloads. In contrast, the HDD option offers cost-effective capacity-optimized storage for less performance-sensitive data like backups, media editing, or long-term archival storage.

[[fsx]] for Windows File Server supports multi-AZ deployments within an AWS Region by automatically replicating data across Availability Zones. This setup ensures data redundancy and availability in case of infrastructure failures.

### Comparison & Anti-Patterns

| Service               | Use Case                                                              | Anti-pattern                                                         |
| --------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------- |
| [[fsx]] for Windows       | High-performance, Windows-native shared file systems                | Running Windows File Servers on [[ec2]] instances                        |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]                   | NFSv4 shared file systems, POSIX compliance, Linux-native workloads | Sharing files between Windows and Linux hosts using SMB                 |
| [[Storage Gateway]]      | On-premises hybrid cloud storage scenarios                           | Using [[fsx]] to replace existing on-premises file servers without proper planning |
| [[Srinivas_Notes/S3|S3]]                    | Object storage, event-driven workflows                              | Storing shared file system data directly in [[Srinivas_Notes/S3|S3]]                        |

Common anti-patterns include attempting to store shared file system data directly in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], managing Windows File Servers on [[ec2]] instances instead of using [[fsx]], and neglecting to account for AD integration when designing solutions.

### [[appsync|Security]] & Governance

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
          "fsx:Describe*",
          "fsx:CreateFileSystem",
          "fsx:DeleteFileSystem"
      ],
      "Resource": "*",
      "Condition": {"StringEquals": {"aws:PrincipalTag/Environment": "production"}}
    }
  ]
}
```
Cross-Account Access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
      "Action": [
          "fsx:Describe*",
          "fsx:CreateFileSystem",
          "fsx:DeleteFileSystem"
      ],
      "Condition": {"StringLike": {"fsx:ResourceTag/Team": "engineering"}},
      "Resource": ["arn:aws:fsx:*:*:file-system/*"]
    }
  ]
}
```
Organization SCPs:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllButProductionFSxAccess",
      "Effect": "Deny",
      "Action": "fsx:*",
      "NotResource": "arn:aws:fsx:*:123456789012:file-system/*",
      "Condition": {"StringNotEquals": {"aws:PrincipalTag/Environment": "production"}}
    }
  ]
}
```

### Performance & Reliability

Throttling Limits:

- Maximum number of file systems per region: 20
- Maximum number of [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|security]] group associations per file system: 10
- Maximum number of initiators (network interfaces) per file system: 100

Exponential Backoff Strategies:

When encountering throttled requests, implement exponential backoff to ensure reliable processing. Example pseudocode:

```python
for attempt in range(max_attempts):
  try:
    # Perform API operation here
    break
  except ThrottlingException:
    if attempt < max_attempts:
      time.sleep(2**attempt * 100ms)
    else:
      raise
```

HA/DR Patterns:

Implement Multi-AZ deployments for increased availability within a single region. For [[dr]], create backup snapshots and copy them to another region. Alternatively, enable continuous backups and configure cross-region replication.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved through resource tagging and usage-based reporting. To calculate costs, consider factors such as file system size, provisioned throughput, and backup [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]].

Example cost calculation:

- File system size: 1 TB (HDD)
- Provisioned throughput: 128 MB/s
- Backup [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]: 1 day

Total monthly cost = (Storage cost + Throughput cost + Backup cost) \* Number of days

### Professional Exam Scenarios

Scenario 1: A company wants to migrate their current Windows-based shared file system to AWS while maintaining the same level of feature compatibility. They need to maintain Active Directory integration and support both SMB and NFSv4 clients. What solution should they adopt?

Correct answer: Implement [[fsx]] for Windows File Server for the Windows-based shared file system requirements and Amazon Elastic File System ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]) for NFSv4 client support. Ensure proper AD integration in both services.

Incorrect answer: Rely solely on [[fsx]] for Windows File Server, which does not offer NFSv4 support.

Scenario 2: An organization needs to deploy a highly available, cost-efficient, and durable file system for storing up to 100 TB of data. They have a strict requirement for 99.9% availability and expect to write data once but read it frequently. Which architecture would you recommend?

Correct answer: Deploy an [[fsx]] for Windows File Server file system with HDD storage configured for high capacity. Enable Multi-AZ deployments to achieve higher availability.

Incorrect answer: Create an [[fsx]] for Windows File Server file system with SSD storage due to insufficient capacity and higher costs.