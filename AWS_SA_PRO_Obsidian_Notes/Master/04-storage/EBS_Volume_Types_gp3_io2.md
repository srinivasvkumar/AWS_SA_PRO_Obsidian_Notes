## Advanced Architecture

### [[ebs|EBS Volume Types]]: gp3 and io2

At their core, General Purpose SSD (gp3) and Provisioned IOPS SSD (io2) are block storage services provided by Amazon Elastic Block Store ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]) for [[ec2]] instances. They differ in performance characteristics, pricing, and ideal workload scenarios.

#### gp3

gp3 volumes offer a baseline of 3,000 IOPS and up to 125 MiB/s throughput with the ability to burst up to 18,000 IOPS and 1,000 MiB/s. gp3 provides a balance between price point and performance, making it suitable for boot volumes, small databases, and development/test environments.

#### io2

io2 volumes offer high baseline IOPS (up to 64,000 IOPS) and throughput (up to 1,000 MiB/s) with predictable performance and consistent low latency. io2 is designed to support IOPS-intensive applications such as large databases, mission-critical applications, and transactional processing systems.

### Under The Hood Mechanics

Both volume types store data in EBS-managed RAID-based storage configurations using dedicated hardware across multiple Availability Zones (AZs) for durability and redundancy. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots create Amazon Machine Images (AMIs) for backup, recovery, and cloning purposes.

## Comparison & Anti-Patterns

| Service         | gp3 Volumes | io2 Volumes |
|-----------------|-------------|-------------|
| Ideal Workloads | Boot volumes, small databases, dev/test envs | Large databases, mission-critical apps, transactional processing sys |
| Max Throughput  | 1,000 MiB/s   | 1,000 MiB/s   |
| Min IOPS       | 100 IOPS     | 1,000 IOPS   |
| Max IOPS       | 16,000 IOPS  | 64,000 IOPS  |
| Cost           | Lower        | Higher      |

### Anti-Patterns

- Using gp3 volumes for IOPS-intensive workloads instead of io2 volumes.
- Failing to properly size gp3 volumes based on desired performance.
- Ignoring io2 volume [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] when designing mission-critical applications.

## [[appsync|Security]] & Governance

Implementing least privilege access, cross-account access, and organization Service Control [[policies]] (SCPs) are essential for securing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] resources.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy Example

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachVolume",
                "ec2:DescribeVolumes",
                "ec2:CreateVolume"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

### Cross-Account Access

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
            },
            "Action": [
                "ec2:AttachVolume",
                "ec2:DescribeVolumes",
                "ec2:CreateVolume"
            ],
            "Resource": "arn:aws:ec2:*::volume/*",
            "Condition": {
                "StringEquals": {
                    "ec2:Owner": "ACCOUNT_ID"
                }
            }
        }
    ]
}
```

### Organization SCPs

Limit actions on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] resources at an organizational level:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "NotAction": [
                "ec2:Describe*",
                "ec2:CreateVolume"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Performance & Reliability

To ensure optimal performance and reliability, implement throttling limits and exponential backoff strategies.

### Throttling Limits

gp3 and io2 have specific API request limits per account:

- gp3: 200 requests/sec
- io2: 200 requests/sec

### Exponential Backoff Strategies

When encountering [[api-gateway|errors]], apply exponential backoff strategies to handle transient failures:

```mermaid
graph LR
A[Request] --> B[Error]
B --> C[Wait 1s]
C --> D[Request]
D -->|Error| E[Wait 2s]
E --> F[Request]
F -->|Error| G[Wait 4s]
G --> H[Request]
H -->|Success| I[Complete]
I --> J[Reset Wait Time]
```

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented using AWS [[billing|Cost Explorer]] or tagging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] resources. Calculate costs using the following formulae:

- gp3: `$0.10 per GB-month of provisioned storage + $0.10 per 1 million IOPS-month`
- io2: `$0.125 per GB-month of provisioned storage + $0.125 per provisioned IOPS-month`

## Professional Exam Scenarios

### Scenario 1

An e-commerce company requires high-performance storage for its databases during peak shopping seasons. Which solution would you recommend?

1. Create io2 volumes with 1,000 IOPS and 100 GiB of storage.
2. Create gp3 volumes with 16,000 IOPS and 500 GiB of storage.
3. Create io2 volumes with 64,000 IOPS and 1 TiB of storage.

Correct Answer: c, as it offers the highest IOPS and storage capacity suitable for IOPS-intensive databases.

### Scenario 2

A media organization needs to store video files for short-term projects while keeping costs minimal. Which solution would you recommend?

1. Create gp3 volumes with 1,000 IOPS and 100 GiB of storage.
2. Create gp3 volumes with 3,000 IOPS and 500 GiB of storage.
3. Create gp3 volumes with 10,000 IOPS and 1 TiB of storage.

Correct Answer: a, as it balances lower IOPS requirements with smaller storage capacities, minimizing overall costs.