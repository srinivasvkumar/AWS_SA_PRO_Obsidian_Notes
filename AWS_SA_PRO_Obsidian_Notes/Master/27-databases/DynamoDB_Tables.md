**Advanced Architecture**

[[dynamodb|DynamoDB Tables]] are a managed NoSQL database service provided by AWS. At its core, [[dynamodb]] uses a partitioned hash table to store data across multiple nodes. Each table has one or more primary keys, which can be either partition key only or a composite of partition key and sort key. Understanding how [[dynamodb]] stores data is crucial for designing performant systems.

When creating a new table, you need to specify its provisioned throughput capacity in terms of read and write capacity units. The total capacity is divided among partitions based on the amount of data stored in your table. As your data grows, [[dynamodb]] automatically adds and removes partitions to maintain optimal performance.

At the global level, [[dynamodb]] supports multi-region replication using Global Tables. This feature allows you to create a single table that replicates data across multiple regions, ensuring low-latency reads and writes worldwide. To achieve high availability and durability, [[dynamodb]] also offers DAX ([[dynamodb]] Accelerator) as an in-memory cache and automatic backups with point-in-time recovery.

**Comparison & Anti-Patterns**

| Service                      | When to use                                                              |
| ---------------------------- | ----------------------------------------------------------------------- |
| [[dynamodb|DynamoDB Tables]]             | High-performance, flexible schema, large datasets                       |
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]                | Relational databases, SQL support, compatibility with popular engines |
| Amazon [[DocumentDB]]         | MongoDB-compatible document database                                   |
| Amazon Keyspace (for Cassandra) | Apache Cassandra-compatible database                           |
| Amazon [[QLDB]]                | Immutable ledger for compliance and auditing purposes                    |

Anti-patterns include storing relational data in [[dynamodb]] without proper normalization, leading to performance issues and increased storage costs. Another common mistake is not adjusting throughput capacity settings according to usage patterns, causing throttling [[api-gateway|errors]].

**[[appsync|Security]] & Governance**

To secure [[dynamodb|DynamoDB tables]], apply complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] at various levels: resource, organization, and account. Here's an example policy granting access to specific actions on a table:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:BatchGetItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:us-west-2:123456789012:table/MyTable"
      ]
    }
  ]
}
```

Cross-account access can be enabled using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. For centralized governance, use Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] to enforce restrictions on member accounts.

**Performance & Reliability**

Throttling limits in [[dynamodb]] depend on provisioned throughput capacity. If you exceed these limits, you'll receive a ProvisionedThroughputExceededException error. To handle such situations, implement exponential backoff strategies with jitter when retrying requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], enable Point-in-Time Recovery (PITR) and distribute data globally using Global Tables. Consider implementing DAX as an in-memory cache for read-intensive workloads.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls in [[dynamodb]] include setting separate capacity units for read and write operations, enabling auto-scaling for dynamic workloads, and utilizing on-demand mode for occasional spikes. Additionally, optimize performance by selecting appropriate attribute types (e.g., string, number, binary) and distributing data evenly across partitions.

Calculate the cost of running a [[dynamodb]] table using the following formula:

Cost = (ReadCapacityUnits \* ReadCapacityUnitPrice) + (WriteCapacityUnits \* WriteCapacityUnitPrice) + Storage \* StoragePrice

**Professional Exam Scenarios**

Scenario 1: A company needs to store customer information in a highly available and scalable manner while meeting strict [[appsync|security]] requirements. They expect millions of users and frequent updates.

Correct Answer: Use [[dynamodb|DynamoDB Tables]] with encryption at rest, enable [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] for private connectivity, and set up fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Implement Global Tables for low-latency reads and writes across regions.

Incorrect Answer: Use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] since it provides SQL support and compatibility with popular engines. While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] offers some benefits, it doesn't provide the same level of scalability and flexibility as [[dynamodb]].

Scenario 2: An e-commerce platform experiences sudden traffic surges during holiday seasons but maintains steady traffic during non-peak times.

Correct Answer: Utilize [[dynamodb]]'s on-demand mode to handle variable traffic patterns without worrying about capacity planning. Enable auto-scaling to minimize costs during non-peak times.

Incorrect Answer: Set fixed provisioned throughput capacities based on peak traffic expectations. This approach may lead to underutilization during non-peak times and wasted resources due to overprovisioning.

### AI Linked Diagram
![[Screenshot 2026-01-01 at 11.42.30 PM.png]]
