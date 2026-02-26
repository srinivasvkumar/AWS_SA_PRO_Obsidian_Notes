## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Aurora]] Multi-Master is a distributed relational database system that allows multiple nodes to accept write workloads concurrently. It achieves this by maintaining identical data copies across multiple Availability Zones (AZs) within an AWS Region.

Internally, [[aurora]] Multi-Master uses a "quorum" of at least three replicas to ensure consistency and fault tolerance. Each AZ in a region hosts one replica, allowing automatic failover and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities without any manual intervention. The following Mermaid syntax diagram illustrates the architecture:

```mermaid
graph LR
A[Primary DB Instance] -->|Write| B[Replica 1]
A -->|Write| C[Replica 2]
A -->|Write| D[Replica 3]
B -->|Read Replicas| E(Read Replicas)
C -->|Read Replicas| E
D -->|Read Replicas| E
classDef node fill:#f9d2d2,stroke:#333,stroke-width:4px;
classDef edge stroke:#333,stroke-width:4px;
class A,B,C,D node;
class E node node/read replicas;
classEdge edge;
```

The underlying mechanism involves a Paxos-based protocol called "[[aurora]] Global Database", which ensures strong consistency among multiple nodes while providing a single view of the data. This protocol enables conflict resolution when writes occur simultaneously across multiple nodes.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

Here's a comparison table between [[aurora]] Multi-Master and other databases:

| Service          | Use Case                                                              |
| ---------------- | -------------------------------------------------------------------- |
| [[aurora]] Multi-Master | High-throughput applications requiring multi-region active-active writes |
| [[dynamodb]] Global Table | NoSQL datastore demanding high throughput and low latency across regions   |
| AWS [[DocumentDB]] | Multi-region read replicas for document databases                     |
| AWS Keyspace (Cassandra) | Multi-region deployments for Cassandra workloads                      |

Anti-patterns include using [[aurora]] Multi-Master as a simple load balancer for read replicas or as a geo-redundant backup solution. These scenarios are better suited for services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas or [[dynamodb|DynamoDB Global Tables]].

## [[RDS_Instance_Types|3. Security & Governance]]

Implementing complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] requires fine-grained control over resource access. For example, you can create a JSON policy snippet like this to allow cross-account access:

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
                "rds-db:connect"
            ],
            "Resource": "arn:aws:rds-data:us-west-2:987654321098:dbuser:mydbname/mymultimasteruser",
            "Condition": {
                "Bool": {
                    "aws:ViaAWSService": "true"
                }
            }
        }
    ]
}
```

To enforce [[appsync|security]] and governance across accounts, leverage Organizational Service Control [[policies]] (SCPs) to restrict specific API calls related to [[aurora]] Multi-Master.

## [[RDS_Instance_Types|4. Performance & Reliability]]

[[aurora]] Multi-Master has built-in throttling limits based on the number of active connections per instance. To mitigate performance degradation due to throttling, implement exponential backoff strategies for retries. For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], use [[aurora]] Global Database to maintain a secondary cluster in another region.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include monitoring usage metrics such as storage consumed, number of instances, and cross-region traffic. Calculate costs using the AWS Pricing Calculator or the AWS [[billing|Cost Explorer]] tool. Implement tagging strategies to monitor usage and allocate costs accurately.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1

Your company operates in two regions: US West (Oregon) and EU West (Ireland). Your application requires a highly available, multi-master database solution capable of handling thousands of transactions per second. Which solution would you recommend?

#### Correct Answer:

Implement [[aurora]] Multi-Master with read replicas in both regions. This configuration offers multi-region support, active-active write capability, and automatic failover for high availability.

#### Incorrect Answers:

1. Using [[dynamodb]] Global Table: While it supports multi-region high availability, it doesn't offer native SQL support required by the application.
2. Implementing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Read Replicas: Although they provide read scaling, they don't natively support active-active write capabilities.

### Scenario 2

You need to secure your [[aurora]] Multi-Master setup across multiple AWS accounts. How do you achieve this?

#### Correct Answer:

Implement a combination of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access rules, and Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]]. By defining the necessary permissions, you can grant secure access to [[aurora]] Multi-Master resources across different accounts.

#### Incorrect Answers:

1. Granting public access via IP whitelisting: This approach does not provide the desired granularity needed for multi-account setups.
2. Sharing resources using AWS Resource Access Manager: While it simplifies sharing resources, it lacks the ability to define fine-grained permission levels.

### AI Linked Diagram
![[Screenshot 2025-12-23 at 4.04.35 PM.png]]


### AI Linked Diagram
![[Screenshot 2025-12-23 at 4.59.54 PM.png]]
