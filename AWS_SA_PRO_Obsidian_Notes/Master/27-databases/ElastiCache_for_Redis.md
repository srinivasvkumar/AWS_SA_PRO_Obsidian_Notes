## Advanced Architecture

[[elasticache]] for Redis is a fully managed in-memory data store service offered by AWS. It provides sub-millisecond latency and high throughput for read-heavy workloads. The internal architecture of [[elasticache]] for Redis includes various components like nodes, clusters, shards, replicas, and cache [[appsync|security]] groups.

### Nodes and Clusters

An [[elasticache]] node is an instance of the Redis software engine. A cluster consists of one or more nodes. Each node within a cluster can be addressed as a separate endpoint. There are two types of nodes:

- **Single-node clusters**: Ideal for development and testing purposes.
- **Multi-node clusters (sharded)**: Designed for production environments that require high availability and scalability. Sharding divides your dataset into multiple nodes based on a hashing algorithm.

### [[RDS_Instance_Types|Global Scale Considerations]]

[[elasticache]] supports multi-region replication using Redis (cross-region replication) and Memcached (using AWS Data Streams and [[lambda]]). Multi-region deployments improve application resilience and reduce latency when serving users from different geographical locations.

#### Mermaid Diagram: Multi-Region Replication
```mermaid
graph TD
    Subnet[Subnets] --> Node[Node(s)]
    CacheCluster[Cache Cluster] --> Subnet
    RouteTable[Route Table] --> Subnet
    
    CacheCluster --> AWSDataStream[AWS Data Stream]
    AWSDataStream --> Lambda[Lambda Function]
    Lambda --> DestinationRegion[Destination Region]
    
    DestinationRegion --> DestinationCacheCluster[Destination Cache Cluster]
    DestinationCacheCluster --> DestinationSubnet[Destination Subnets]
    DestinationSubnet --> DestinationNode[Node(s)]
```

## Comparison & Anti-Patterns

| Service                 | Use Case                                         | Anti-pattern                                |
|-------------------------|---------------------------------------------------|----------------------------------------------|
| [[elasticache]] for Redis   | Real-time applications, [[api-gateway|caching]], session management | Inappropriate use for persistent storage      |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] MySQL              | Persistent relational database                   | Overuse for ephemeral [[api-gateway|caching]]                 |
| Amazon [[dynamodb]]        | Highly available NoSQL database                   | Overuse for in-memory [[api-gateway|caching]]                |

## [[appsync|Security]] & Governance

Implementing least privilege access involves creating fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and applying Service Control [[policies]] (SCPs) at the organization level.

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy example allowing a user to create and manage [[elasticache]] resources but not delete them:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "elasticache:Create*",
        "elasticache:Describe*",
        "elasticache:Modify*",
        "elasticache:Authorize*",
        "elasticache:Revoke*",
        "elasticache:Test*",
        "elasticache:List*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "elasticache:Delete*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Cross-Account Access

To enable cross-account access, you need to set up resource-based [[policies]] on [[elasticache]] resources and grant permissions via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Here's an example of a resource-based policy allowing a specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role from another account to perform the `elasticache:ModifyCacheParameterGroup` action on a Redis parameter group:

```json
{
  "Version": "2012-10-17",
  "Id": "example-policy",
  "Statement": [
    {
      "Sid": "ExampleStmt",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/example-role"
      },
      "Action": "elasticache:ModifyCacheParameterGroup",
      "Resource": "arn:aws:elasticache:us-west-2:012345678901:parameter-group/example-param-group",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:iam::123456789012:role/example-role"
        }
      }
    }
  ]
}
```

## Performance & Reliability

Throttling limits depend on the type of operation and the number of nodes. For example, `CreateCacheCluster` has a limit of 10 requests per minute. To handle throttled requests, implement exponential backoff strategies with jitter.

High Availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] ([[dr]]) patterns include Multi-AZ deployments, backup and restore, and point-in-time recovery. Ensure appropriate configuration of cache [[appsync|security]] groups and cache node endpoints to achieve optimal performance and reliability.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls involve monitoring and adjusting parameters such as cache node size, usage duration, and cache [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. Additionally, optimize costs by implementing proper capacity planning and right-sizing techniques.

For example, if you have a single-node cluster with a cache.t3.small node size and daily cache utilization peaks around 4 GB, you could upgrade to a cache.t3.medium node size with 8 GB memory. This would provide headroom for future growth and prevent potential performance degradation while keeping costs low.

## Professional Exam Scenarios

### Scenario 1: Multi-Account Strategy

Suppose you want to deploy a highly available [[elasticache]] for Redis cluster across multiple AWS accounts for governance reasons. Which solutions should you choose?

Correct answer: Implement cross-account access by setting up resource-based [[policies]] on [[elasticache]] resources and granting permissions via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. This way, you can ensure secure communication between the [[elasticache]] clusters in different accounts.

Incorrect answer: Create [[elasticache]] clusters in each account without any interconnectivity. This approach does not allow efficient communication between the [[elasticache]] clusters.

### Scenario 2: Cost Management

You notice increasing costs related to [[elasticache]] services due to unpredictable traffic patterns. How do you optimize costs without compromising performance?

Correct answer: Implement granular cost controls by monitoring and adjusting parameters such as cache node size, usage duration, and cache [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. Perform capacity planning and right-size cache nodes based on traffic analysis.

Incorrect answer: Downgrade all cache nodes to smaller sizes without analyzing traffic patterns. This approach may compromise performance during peak traffic times.