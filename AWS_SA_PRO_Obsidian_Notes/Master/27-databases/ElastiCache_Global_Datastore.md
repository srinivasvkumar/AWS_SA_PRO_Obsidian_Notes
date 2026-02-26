## Advanced Architecture

At its core, Amazon [[elasticache]] Global Datastore is a fully managed, highly scalable, and high-performing in-memory data store service that replicates data across multiple regions for low-latency reads and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities. It uses either Redis or Memcached as the underlying engine. The key differentiator of Global Datastore is its ability to automatically shard data, manage replication, and provide automatic failover at a global level.

Internally, Global Datstore builds upon [[elasticache]] clusters in multiple regions, synchronously replicating data between them using Redis' native replication protocol. This ensures a consistent view of data while minimizing latency for read operations. For higher availability, [[elasticache]] Global Datastore supports multi-AZ replication within a single region. In case of an outage in one region, clients can automatically connect to replica nodes in another region.

Global Datastore also offers advanced features like automated conflict resolution, which handles updates made to the same data item concurrently in different regions. To achieve this, it uses a "Last Writer Wins" approach based on timestamps.

## Comparison & Anti-Patterns

| Service             | Use Case                                                          | Anti-pattern                                                        |
| ------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------- |
| [[elasticache]] Global  | Multi-region, low-latency reads, write-heavy workloads            | Single-region applications, [[api-gateway|caching]] static content                |
| [[dynamodb]] Global     | Multi-region, eventual consistency, write-scalable workloads      | Low latency requirements, locality-based queries                   |
| Database Migration  | Homogeneous DB migrations, refreshable read replicas              | Real-time cross-region writes, multi-master setups                |

Common design mistakes include using Global Datastore for simple [[api-gateway|caching]] scenarios or when eventual consistency is acceptable.

## [[appsync|Security]] & Governance

To secure access to [[elasticache]] Global Datastore, you need to implement proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access mechanisms, and organization Service Control [[policies]] (SCPs).

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy Example

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticache:Describe*",
                "elasticache:Modify*",
                "elasticache:Create*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

### Cross-Account Access

To enable cross-account access, create a role in the source account allowing the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles to perform specific actions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::DESTINATION_ACCOUNT_ID:root"
            },
            "Action": [
                "elasticache:Describe*",
                "elasticache:Modify*",
                "elasticache:Create*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "vpc-SOME_VPC_ID"
                }
            }
        }
    ]
}
```

### Organization SCPs

Use Organization SCPs to enforce restrictions on creating and configuring [[elasticache]] resources. For example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "elasticache:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIgnoreCase": {
                    "aws:PrincipalTag/Team": "DataServices"
                }
            }
        }
    ]
}
```

## Performance & Reliability

[[elasticache]] Global Datastore has throttling limits depending on the instance type and region. When exceeded, consider implementing exponential backoff strategies to avoid hitting these limits continuously.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], follow these [[iam|best practices]]:

* Enable auto-discovery to automatically switch to healthy nodes during failures.
* Implement [[route53|health checks]] and fallbacks to other regions if needed.
* Monitor cache metrics through [[cloudwatch]] to detect potential issues early.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls involve understanding the costs associated with instance types, regions, and usage hours. Calculate the estimated monthly costs by considering the number of nodes, their size, and region-specific pricing.

Additionally, monitor your utilization rates and downsize instances when possible. Consider using reserved nodes for predictable workloads.

## Professional Exam Scenarios

Scenario 1:
You are designing a multi-region [[iot]] platform using AWS services including [[kinesis]], [[lambda]], [[dynamodb]], and [[elasticache]] Global Datastore. The system must process millions of events per second while maintaining low latency. What caveats should be considered when choosing [[elasticache]] Global Datastore over [[dynamodb]]?

Correct answer: Since [[elasticache]] Global Datastore relies on Redis or Memcached engines, it may not natively support the same write scaling capabilities as [[dynamodb]]. Therefore, carefully evaluate the expected write load to ensure [[elasticache]] Global Datastore can handle it efficiently without compromising performance.

Incorrect answer: Assuming [[dynamodb]] always provides lower latency than [[elasticache]] Global Datastore because it's a NoSQL database.

Scenario 2:
A company wants to share [[elasticache]] Global Datastore resources between two accounts for cross-region replication purposes. How would you implement this?

Correct answer: Create a role in the source account granting permissions to the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles. Then, reference this role in the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy to allow access to [[elasticache]] Global Datastore resources.

Incorrect answer: Creating a shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] between both accounts and placing [[elasticache]] Global Datastore resources inside that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].