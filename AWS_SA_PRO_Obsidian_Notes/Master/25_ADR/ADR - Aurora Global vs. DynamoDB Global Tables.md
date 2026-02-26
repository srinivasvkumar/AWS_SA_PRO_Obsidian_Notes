```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
priority: high
---

> [!abstract] Exam Tip: When preparing for the SAP-C02 exam, it's crucial to understand both Aurora Global Database and DynamoDB Global Tables in detail. Each has its unique features, use cases, and considerations.

## Under-the-Hood Mechanics

### Aurora Global Database

- **Data Flow**: Writes are performed in the primary region and asynchronously replicated to up to five secondary regions.
- **Consistency Models**:
  - **Read Consistency**: Strongly consistent reads within the primary region, eventually consistent reads from replicas by default. Can be configured for strong consistency with higher latency in secondary regions.
  - **Global Data Access**: Supports read-only queries and failover mechanisms in secondary regions.

### DynamoDB Global Tables

- **Data Flow**: Uses AWS-managed replication across multiple regions; writes are performed in the primary region and asynchronously replicated to other regions.
- **Consistency Models**:
  - **Strongly Consistent Reads**: Available within the primary region, eventually consistent reads from secondary regions by default. Conditional updates support local consistency in secondary regions.

## Exhaustive Feature List

### Aurora Global Database

1. **Replication**: Asynchronous cross-region replication.
2. **Read Replicas**: Multiple read replicas supported both in primary and secondary regions.
3. **Failover Mechanism**: Automatic failover to secondary region during primary failure.
4. **Backup and Restore**: Cross-region backups and restores are supported.
5. **Security**: VPC integration, IAM roles for encryption, AWS Secrets Manager.

### DynamoDB Global Tables

1. **Replication**: Asynchronous cross-region replication.
2. **Read Replicas**: Read-only access in secondary regions; supports conditional updates based on local consistency.
3. **Failover Mechanism**: Conditional writes and read-after-write consistency support.
4. **Backup and Restore**: Point-in-time recovery (PITR) for all tables.
5. **Security**: IAM roles, KMS encryption.

## Implementation

### Aurora Global Database

1. **Initial Configuration**: Create Aurora cluster in the primary region.
2. **Setup Replicas**: Add replicas in secondary regions.
3. **Enable Failover**: Configure failover settings for disaster recovery.
4. **Multi-Account Governance**: Use IAM roles, SCPs (Service Control Policies), and RAM to manage cross-account access.

### DynamoDB Global Tables

1. **Initial Configuration**: Create DynamoDB table in the primary region.
2. **Setup Replicas**: Enable global tables feature to add replicas in other regions.
3. **Configure Consistency**: Set read/write consistency levels as needed.
4. **Multi-Account Governance**: Use IAM roles, SCPs, and RAM for cross-account access.

## Technical Limits (2026)

### Aurora Global Database

- Primary Region: 15 TB storage
- Secondary Regions: Up to five regions with up to 15 TB each.
- Adjustable: Storage capacity can be increased.
- Non-adjustable: Number of secondary regions is capped at five.

### DynamoDB Global Tables

- Maximum number of global tables per account: Unlimited (subject to service limits).
- Maximum number of regional tables per global table: 8.
- Adjustable: Throughput and storage capacity are adjustable.
- Non-adjustable: Fixed maximum of 8 regional tables per global table.

## CLI & API Grounding

### Aurora Global Database

- **Create Aurora Cluster**: `aws rds create-db-cluster`
- **Add Replicas**: `aws rds add-db-cluster-members`

### DynamoDB Global Tables

- **Enable Global Table**: `aws dynamodb update-global-table-settings`
- **Describe Table Settings**: `aws dynamodb describe-global-table-settings`

## Pricing & TCO

### Aurora Global Database

- Cost drivers: Storage, compute hours, and cross-region replication.
- Optimization strategies: Utilize Reserved Instances (RIs), Savings Plans.

### DynamoDB Global Tables

- Cost drivers: Read/write capacity units (RCUs/WCUs).
- Optimization strategies: On-demand capacity mode for unpredictable workloads, reserved capacity for steady-state usage.

## Competitor Comparison

### Aurora vs. DynamoDB

- **Scalability**: Aurora scales vertically and horizontally; DynamoDB auto-scales.
- **Consistency Models**: Aurora supports strong consistency with eventual consistency options; DynamoDB offers eventually consistent reads by default.
- **Use Cases**: Aurora is ideal for relational databases requiring strong consistency, while DynamoDB excels in NoSQL use cases needing high scalability.

## Contextualization within AWS Ecosystem

### Aurora Global Database

- Integrates with RDS, VPC, IAM, and CloudWatch for monitoring and security.
- Fits into microservices architectures where consistent data access across regions is critical.

### DynamoDB Global Tables

- Works well in serverless architectures using Lambda, API Gateway.
- Ideal for event-driven architectures needing global consistency.

## Real-World Use Cases

### Aurora Global Database

- Multi-region e-commerce application with strong transactional requirements.
- Financial services requiring high availability and disaster recovery across regions.

### DynamoDB Global Tables

- High-throughput IoT applications collecting data from multiple regions.
- Social media platforms needing consistent global user experience.

## Exam Traps & Common Misconceptions

### Aurora Global Database

1. **Misunderstanding Failover Latency**: Not understanding the latency impacts of failovers and eventual consistency in secondary regions.
2. **RPO/RTO Confusion**: Aurora may not be the best choice if your RPO is extremely low, as it supports eventual consistency.

### DynamoDB Global Tables

1. **Eventual Consistency Overlooked**: Eventual consistency can lead to data discrepancies between primary and secondary regions.
2. **Cross-Account Access Complexities**: Misconfigurations in IAM roles and SCPs can lead to access issues during cross-account migrations.

## Decision Matrix Format

| Feature              | Aurora Global Database             | DynamoDB Global Tables            |
|----------------------|------------------------------------|-----------------------------------|
| Replication          | Asynchronous                       | Asynchronous                      |
| Read Consistency     | Strongly consistent (primary), configurable strong consistency in secondary regions | Eventual (secondary) with conditional updates   |
| Failover Mechanism   | Automatic                           | Conditional writes                |
| Backup & Restore     | Cross-region                       | Point-in-time recovery            |

## Service Updates and Exam Focus

- **Latest 2026 AWS Updates**: Enhanced security features, improved failover mechanisms.
- **Exam Patterns**: Common scenarios include multi-region disaster recovery configurations.

## Architectural Trade-offs

### Aurora Global Database

- Pros: Strong consistency in primary region, robust transactional support.
- Cons: Higher latency for reads from secondary regions, limited scalability compared to DynamoDB.

### DynamoDB Global Tables

- Pros: High scalability, low-cost read/write operations.
- Cons: Eventual consistency in secondary regions, fewer features compared to Aurora (e.g., transactions).

## Enterprise Governance Layers

1. **AWS Organizations**: Use AWS Organizations to manage multiple accounts and apply SCPs for policy enforcement.
2. **Service Control Policies (SCPs)**: Apply SCPs to restrict access to specific services or operations across the organization.
3. **Resource Access Manager (RAM)**: Share resources like VPC endpoints, security groups across accounts within the organization.
4. **CloudTrail Auditing**: Enable CloudTrail for auditing and compliance, ensuring all changes are tracked and logged.

## Complex Failure Modes

### Aurora Global Database

- **KMS Key Policy Conflicts in Cross-Account Migrations**: Misconfigurations in KMS key policies can prevent cross-account replication, leading to data inconsistency.
- **Replication Lag**: High write volumes can cause significant lag between primary and secondary regions.

### DynamoDB Global Tables

- **Data Consistency Issues Across Regions**: Due to eventual consistency, reads might return outdated data until replication completes.
- **IAM Role Misconfigurations**: Incorrect IAM roles or policies may result in access issues during cross-account operations.

## Summary

This enhanced review covers Aurora Global Database and DynamoDB Global Tables with a focus on under-the-hood mechanics, exhaustive feature lists, implementation steps, technical limits, CLI & API grounding, pricing & TCO, competitor comparisons, real-world use cases, architectural diagrams, exam traps, decision matrices, latest service updates, architectural trade-offs, enterprise governance layers, complex failure modes. The content is structured to provide comprehensive preparation for the SAP-C02 exam.

### Related Notes

- [[AWS RDS]]
- [[DynamoDB]]
- [[Aurora Multi-AZ]]
- [[Global Application Deployment]]
```