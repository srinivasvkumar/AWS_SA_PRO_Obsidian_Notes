### [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Keyspaces]] (Cassandra) is a scalable, highly available NoSQL database service that is fully compatible with Apache Cassandra. It allows you to reuse your existing Cassandra data models and provides a seamless transition from operational databases to a managed database service.

Internally, Keyspaces uses a partitioned key-space design with virtual nodes (vNodes) for even distribution of data and load across multiple nodes. This architecture enables linear scalability while maintaining high availability and fault tolerance.

Keyspaces supports global tables, allowing you to distribute data across multiple regions for low-latency read and write operations. Data in global tables is automatically replicated to all regions you select, providing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities out of the box.

### [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service           | Partition Key   | Scalability      | Data Model    | Time-series data |
|-------------------|:--------------:|:---------------:|:-------------:|:---------------:|
| Keyspaces (Cassandra) | Required | Excellent | Complex | Limited          |
| [[dynamodb]]         | Required       | Excellent        | Simple        | Very good        |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (MySQL/PostgreSQL) | Not Applicable | Good            | Complex       | Poor             |

*Common design mistakes include:*

- Failing to define an appropriate partition key strategy leading to hotspots and uneven data distribution.
- Using Cassandra data models without understanding their implications in the context of Keyspaces.

### [[RDS_Instance_Types|3. Security & Governance]]

To manage [[appsync|security]] and governance in Keyspaces, you can create custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] based on specific needs. For example, granting access to specific keyspaces or resources using resource-based [[policies]].

Cross-account access can be achieved by creating a resource-based policy in the destination account that grants permissions to the source account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role. Additionally, you can enforce centralized control over AWS services using Service Control [[policies]] (SCPs) at the organization level.

#### JSON Snippet Example
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/keyspace-access"},
      "Action": ["cassandra:DescribeKeyspace", "cassandra:Query"],
      "Resource": "arn:aws:keyspaces:us-west-2:012345678901:keyspace/my_keyspace/*"
    }
  ]
}
```

### [[RDS_Instance_Types|4. Performance & Reliability]]

Keyspaces has built-in throttling limits to prevent abuse and ensure fair usage among customers. To handle throttled requests, implement exponential backoff strategies using the AWS SDKs.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], use multi-region deployments with global tables. Implementing automatic failover and configuring read and write endpoints ensures minimal downtime during failures.

### [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls in Keyspaces are primarily achieved through on-demand capacity mode, which charges for the resources consumed per second. In addition, provisioned capacity mode allows reserving resources for predictable workloads, offering up to a 65% discount compared to on-demand pricing.

Calculation Examples:

- On-Demand Capacity Mode: `(Read/Write Requests * Operations Per Second Rate) + Storage Used`
- Provisioned Capacity Mode: `(Number of Nodes * NodeHours) * Provisions Price`

### [[RDS_Instance_Types|6. Professional Exam Scenarios]]

#### Scenario 1

Your company wants to migrate its existing Apache Cassandra clusters to a fully managed service. The solution must support multi-region deployments and provide low-latency read and write operations. Which AWS service should you recommend?

*Correct Answer*: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Keyspaces]] (Cassandra)
*Justification*: Keyspaces offers compatibility with Apache Cassandra, enabling seamless migration and integration with existing data models. Moreover, it supports multi-region deployments via global tables for low-latency operations.

#### Scenario 2

Your organization manages multiple AWS accounts under a single [[AWS Organization]]. You need to restrict the creation of new Keyspaces instances to specific teams within these accounts. How would you implement this using AWS Services?

*Correct Answer*: Implement Service Control [[policies]] (SCPs) at the organization level.
*Justification*: SCPs allow enforcing centralized control over AWS services, preventing unauthorized changes or actions. By defining an [[SCP]] that restricts the creation of Keyspaces resources, you can limit the scope to specific teams within the accounts.