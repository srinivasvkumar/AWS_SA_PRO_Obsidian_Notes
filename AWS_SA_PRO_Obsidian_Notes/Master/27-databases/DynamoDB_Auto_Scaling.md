**Advanced Architecture**

[[dynamodb]] Auto Scaling operates at the table and index level, using four resources: [[dynamodb]] Table, [[dynamodb]] Global Table, DAX ([[dynamodb]] Accelerator), and [[dms]] (Database Migration Service). Understanding their roles is crucial for advanced architecture:

- Tables: Basic data container with attributes (key, sort key, and optional attributes).
- Global Tables: Multi-region, distributed tables that provide a single endpoint and data replication.
- DAX: In-memory cache that reduces response times for millisecond-bound applications.
- [[dms]]: Tool to migrate data between databases and perform homogeneous or heterogeneous migrations.

Internally, [[dynamodb]] Auto Scaling uses two components: [[dynamodb]] Table Auto Scaling and [[dynamodb]] Global Table Auto Scaling. The former scales storage and throughput (read and write capacity units) based on demand, while the latter synchronizes provisioned throughput across global tables.

The under-the-hood mechanics include three primary components: predictive scaling, dynamic scaling, and target tracking scaling. Predictive scaling uses machine learning algorithms to analyze historical traffic patterns and adjust capacity levels before traffic peaks. Dynamic scaling responds to rapid changes in traffic by adding or removing capacity in small increments. Target tracking scaling maintains a specified minimum or maximum throughput level by automatically adjusting capacity as needed.

**Comparison & Anti-Patterns**

| Service          | Use Case                                                              | Anti-pattern                                                        |
| ---------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------- |
| [[dynamodb]]         | Highly available, flexible schema, unpredictable traffic           | Strict schema requirements, predictable traffic                   |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (relational) | SQL compatibility, transactions, consistency, relationships           | No schema flexibility, high availability not required             |
| [[DocumentDB]]       | Migrating from MongoDB, storing JSON documents                      | Real-time analytics, low latency                                    |
| Keyspace (Cassandra) | Time-series data, [[iot]] telemetry, wide columns                     | Key-value store use cases, relational database use cases            |

Common design mistakes involve choosing the wrong service for the use case, such as using [[dynamodb]] when strict schema requirements are necessary. Another mistake is setting up auto-scaling rules without considering the cost implications.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON code snippets like this one:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:DescribeTable",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:GetItem",
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

Cross-account access involves creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A allowing specific actions on [[dynamodb]] resources in Account B. Organizational Service Control [[policies]] (SCPs) limit permissions at the organization level.

**Performance & Reliability**

Throttling limits in [[dynamodb]] depend on the chosen read and write capacity modes. Exponential backoff strategies should be employed when throttled requests are detected. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include using Global Tables and Point-in-Time Recovery (PITR).

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting individual capacity unit costs per table or index, enabling on-demand capacity mode, and applying usage-based [[cost-allocation-tags|cost allocation tags]]. Calculation examples:

- Provisioned capacity: $0.0000126 x (read\_capacity\_units \* 60 + write\_capacity\_units \* 60) x hours
- On-demand capacity: (number of request units) x hourly rate

**Professional Exam Scenarios**

Scenario 1:
A company requires a highly available, globally distributed database for its mobile app. They expect sudden spikes in traffic during promotions. What solution would you recommend?

Correct answer: Implement a [[dynamodb]] Global Table with Auto Scaling configured for both read and write capacity units. Enable PITR for backup purposes and configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to allow cross-account access for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].

Incorrect answers: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] since it doesn't support automatic scaling; using [[dynamodb]] without Auto Scaling since it wouldn't handle sudden spikes in traffic efficiently.

Scenario 2:
A gaming company needs to store user scores and game events in real-time, with thousands of requests per second. Which service and architecture would best fit this scenario?

Correct answer: Use [[dynamodb]] with DAX for [[api-gateway|caching]] and Auto Scaling enabled. Configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for fine-grained access control and implement exponential backoff strategies for handling throttled requests.

Incorrect answers: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] due to its limited scalability; using [[dynamodb]] without DAX because it may introduce latency for millisecond-bound applications.