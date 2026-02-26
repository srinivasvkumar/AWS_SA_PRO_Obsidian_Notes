Advanced Architecture
---------------------

At its core, [[dynamodb]] is a managed NoSQL database that offers single-digit millisecond performance at any scale. It achieves this by using a partitioned hash table internally, which evenly distributes data across multiple nodes. Each node handles a portion of the overall dataset, called a "partition." Partitions are allocated based on throughput requirements and data volume.

When designing for global scale, [[dynamodb|DynamoDB Global Tables]] provide a seamless way to replicate data across multiple regions. This feature uses automatic conflict resolution and synchronous data writes. The underlying mechanism involves two types of endpoints: administrative and regional. Administrative endpoints allow management of tables, while regional endpoints enable read/write operations. Data is stored in multiple AWS Regions, allowing low-latency access from anywhere in the world.

Comparison & Anti-Patterns
---------------------------

| Service            | Use Case                                                                   | Anti-Pattern                                                              |
| ------------------ | ------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [[dynamodb]]           | High-performance, flexible schema, large datasets                         | Small datasets or transactional workloads better suited for SQL databases |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]                | Relational databases, SQL compatibility                                    | No support for flexible schema                                             |
| [[DocumentDB]]         | Managed MongoDB-compatible document store                                 | Real-time analytics or graph processing                                   |
| Keyspace (in Redis) | In-memory data storage, [[api-gateway|caching]], real-time applications                     | Persistent workload requiring disk storage                               |
| [[elasticache]] (in Memcached) | [[api-gateway|Caching]], session management                                          | Large datasets or complex querying                                      |

[[appsync|Security]] & Governance
----------------------

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be used to grant fine-grained access to [[dynamodb]] resources. Here's an example policy JSON snippet that allows a user to perform read and write actions on a specific table:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem"
            ],
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/MyTable"
        }
    ]
}
```

Cross-account access can be achieved using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and resource-based [[policies]]. For instance, you may create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A that grants read/write permissions to [[dynamodb|DynamoDB tables]] in Account B. Users in Account B can then assume the role in Account A to interact with those tables.

Organization Service Control [[policies]] (SCPs) can enforce restrictions on [[dynamodb]] usage at scale. An example [[SCP]] might disallow creating new [[dynamodb|DynamoDB tables]] within an entire organization unless approved by central IT.

Performance & Reliability
---------------------------

[[dynamodb]] throttles requests when the requested provisioned throughput is exceeded. To handle such situations, implement exponential backoff strategies in your application code. This approach reduces request rates during throttling and increases latency variation, ensuring high availability.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], [[dynamodb]] supports multi-region replication through Global Tables. Additionally, [[dynamodb]] provides point-in-time recovery, which enables backup and restore capabilities at the click of a button.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls include setting separate read and write capacity units for each table. Furthermore, [[dynamodb]] Auto Scaling automatically adjusts capacity based on traffic patterns, further optimizing costs.

Calculate [[dynamodb]] costs as follows:

1. Determine provisioned throughput (read and write capacity units).
2. Estimate data storage requirements (GB-months).
3. Calculate data transfer costs (e.g., API calls, data transfer out to internet).
4. Factor in additional features like DAX, Streams, and Backup/Restore.

Professional Exam Scenarios
----------------------------

Scenario 1:

A company has a multi-account setup with three AWS accounts: A, B, and C. They want to share data between these accounts using [[dynamodb]]. How would you achieve this?

Correct Answer: Implement cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and resource-based [[policies]]. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A that grants read/write permissions to [[dynamodb|DynamoDB tables]] in Account A. Users in Account B and C can then assume the role in Account A to interact with those tables.

Incorrect Answer: Using [[dynamodb]] data export/import functionality to move data between accounts is not the best solution because it requires manual intervention and does not offer real-time sharing capabilities.

Scenario 2:

An e-commerce platform running on AWS needs to ensure their [[dynamodb]] implementation can handle massive spikes in traffic during holiday seasons. What steps should they take to prepare for this increased load?

Correct Answer: Enable [[dynamodb]] Auto Scaling to automatically adjust capacity based on traffic patterns. This will minimize overprovisioning and associated costs. Additionally, optimize indexing and querying strategies to reduce the number of read/write operations required.

Incorrect Answer: Setting up standby instances or manually increasing provisioned capacity beforehand is not ideal since it may lead to idle resources during non-peak times and could result in underprovisioning due to unpredictable traffic patterns.