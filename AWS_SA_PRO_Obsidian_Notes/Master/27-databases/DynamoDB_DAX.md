**Advanced Architecture**

DAX ([[dynamodb]] Accelerator) is an in-memory cache that sits between applications and [[dynamodb|DynamoDB tables]]. It reduces the latency of read-intensive workloads by storing frequently accessed data in memory. The underlying architecture consists of a cluster of cache nodes (minimum three for high availability) and a coordinator node that manages client requests and node membership.

Data is automatically partitioned across multiple nodes based on its primary key. Each node maintains its own cache, replicating data from other nodes using a distributed hash table (DHT). This ensures efficient data distribution while minimizing the impact of single-node failures.

For global scale, DAX supports multi-region replication through DAX clusters in different regions. Changes propagate asynchronously between clusters, allowing low-latency reads at the expense of eventual consistency.

**Comparison & Anti-Patterns**

| Service | Use Case |
|---|---|
| DAX | High-performance, read-intensive workloads where predictable single-digit millisecond latency is required. |
| [[dynamodb]] | General purpose NoSQL database suitable for most use cases. Provides built-in redundancy and performance at scale. |
| [[elasticache]] | In-memory [[api-gateway|caching]] solution for read-heavy workloads that can benefit from lower latency than disk-based storage. Not limited to [[dynamodb]] integration. |

Anti-patterns include write-heavy workloads or those requiring strong consistency, as DAX does not support writes directly and operates in a eventually consistent manner.

**[[appsync|Security]] & Governance**

Secure DAX using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for fine-grained control over who can manage and interact with DAX resources. Example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dax:CreateCluster",
                "dax:DeleteCluster"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "dax:DescribeClusters"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access requires creating a DAX cluster within the account needing access, then granting permissions via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or resource [[policies]]. For stricter governance, apply Service Control [[policies]] (SCPs) at the organization level to limit unwanted changes.

**Performance & Reliability**

DAX throttles requests when capacity limits are exceeded. To handle such situations, implement exponential backoff strategies to retry failed requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute DAX clusters across multiple Availability Zones (AZs) and enable automatic backups. Optionally, create manual backups and restore them to a secondary cluster for point-in-time recovery.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting up [[billing]] alarms based on usage thresholds and configuring Data Transfer costs for specific use cases. Calculate costs using the pricing calculator or [[billing|Cost Explorer]] tool.

**Professional Exam Scenarios**

Scenario 1: A media company needs to serve articles from a [[dynamodb]] table with predictable single-digit millisecond latency during peak hours. They also require a backup strategy for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes.

Correct answer: Implement a DAX cluster in front of the [[dynamodb]] table to reduce latency and improve performance. Enable automatic backups on the DAX cluster and configure cross-region replication for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].

Incorrect answer: Use [[elasticache]] because it provides in-memory [[api-gateway|caching]]. While true, [[elasticache]] doesn't integrate directly with [[dynamodb]] like DAX does.

Scenario 2: An e-commerce platform experiences heavy write traffic on their order [[dynamodb]] table. They want to ensure high availability and prevent data loss.

Correct answer: Since DAX doesn't support write-heavy workloads, use [[dynamodb]] Streams to capture table activity and send updates to a second table for backup purposes. Implement auto-scaling [[policies]] to adjust provisioned throughput based on demand.

Incorrect answer: Implement a DAX cluster in front of the [[dynamodb]] table to increase performance. While this may improve read performance, it won't address the write-heavy nature of the workload nor prevent data loss.