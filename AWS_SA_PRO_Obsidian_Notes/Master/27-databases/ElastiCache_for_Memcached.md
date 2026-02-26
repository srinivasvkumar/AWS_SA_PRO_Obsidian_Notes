## Advanced Architecture

### [[RDS_Instance_Types|Internals]]
Memcached is a popular open-source, high-performance memory object [[api-gateway|caching]] system. [[elasticache]] for Memcached supports two types of nodes: `single-node` and `multi-node`. The latter provides replication within a single node group for increased availability and durability.

The **data distribution mechanism** in [[elasticache]] for Memcached uses consistent hashing, which ensures minimal remapping of keys when adding or removing nodes. This results in lower overhead during scaling operations.

### [[RDS_Instance_Types|Global Scale Considerations]]

To build a globally distributed cache using [[elasticache]] for Memcached, you can implement an active-passive strategy with [[route53|latency-based routing]]. For instance, you could deploy Memcached clusters in multiple regions and configure DNS failover using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]. However, it's important to note that [[elasticache]] does not natively support automatic conflict resolution or data synchronization between clusters.

## Comparison & Anti-Patterns

| Service              | Use Cases                                                                                   | Anti-patterns                                                                         |
| -------------------- | ----------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **[[elasticache]] for Memcached** | High throughput read-heavy workloads, session [[api-gateway|caching]], database query result [[api-gateway|caching]] | Multi-region write-intensive applications without custom conflict resolution mechanisms |
| **[[dynamodb|DynamoDB Accelerator (DAX)]]**    | Low-latency read-heavy workloads requiring predictable performance, large [[dynamodb|DynamoDB tables]] | Decreased write capacity due to DAX's read-only nature, small [[dynamodb|DynamoDB tables]]          |

## [[appsync|Security]] & Governance

Implementing cross-account access involves creating a role in the source account with permissions to perform necessary actions. Below is an example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy allowing [[elasticache]] actions in the destination account:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "elasticache:DescribeCacheClusters",
        "elasticache:CreateCacheCluster",
        "elasticache:ModifyCacheCluster"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

Additionally, [[organizations]] can enforce [[appsync|security]] and governance [[policies]] using Service Control [[policies]] (SCPs) at the organization level.

## Performance & Reliability

[[elasticache]] for Memcached supports throttling limits based on the number of requests per second. To manage these effectively, you should implement exponential backoff strategies using SDKs, libraries, or tools like AWS SDKs, Netflix OSS, or Gremlin.

For high availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]), consider implementing multi-AZ replication with asynchronous replica updates. In case of failure, the standby replica takes over automatically.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include selecting appropriate node sizes and configurations based on your requirements. Additionally, monitor usage metrics such as Evictions, CurrItems, and GetMissedRate to optimize your cache configuration.

## Professional Exam Scenario Questions

1. Question: Your company operates multiple accounts and wants to share a Memcached cluster across them. How would you set up cross-account access?

Correct Answer: Create a role in the source account granting permission to perform [[elasticache]] actions in the destination account. Then, assume the role from the destination account to create, modify, or delete resources.

Incorrect Answer: Implement cross-account sharing by configuring resource-based [[policies]] directly on the [[elasticache]] resources.

Justification: The first answer follows [[iam|best practices]] by using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for cross-account access. The second answer is incorrect because [[elasticache]] does not support resource-based [[policies]].

1. Question: Your application requires a globally distributed cache with low latency. What solution would you recommend, and why?

Correct Answer: Implement an active-passive strategy with [[route53|latency-based routing]] using [[elasticache]] for Memcached in multiple regions.

Incorrect Answer: Utilize [[dynamodb|DynamoDB Accelerator (DAX)]] since it offers better performance than [[elasticache]].

Justification: Although both solutions provide [[api-gateway|caching]] capabilities, the question asks for a globally distributed cache. While [[dynamodb|DynamoDB Accelerator (DAX)]] improves performance for [[dynamodb|DynamoDB tables]], it doesn't inherently support multi-region deployments like [[elasticache]]. Therefore, the recommended approach is using [[elasticache]] with a [[route53|latency-based routing]] strategy.