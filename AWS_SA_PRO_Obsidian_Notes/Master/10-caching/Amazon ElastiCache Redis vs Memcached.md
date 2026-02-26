```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Amazon ElastiCache]]: [[Redis]] vs [[Memcached]]

#### UNDER-THE-HOOD MECHANICS:
**[[Redis]]:**
- **Data Storage and Retrieval**: [[Redis]] stores data in memory, supporting various data structures like strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, and geospatial indexes. Data is stored in key-value pairs.
- **Consistency Models**: [[Redis]] offers strong consistency for single-key operations due to its single-threaded architecture. For multi-key transactions, it uses a locking mechanism.
- **Persistence Options**: [[Redis]] supports RDB (point-in-time snapshotting) and AOF (append-only file) for persistence. This ensures data durability even in case of failures.

**[[Memcached]]:**
- **Data Storage and Retrieval**: [[Memcached]] stores only simple key-value pairs, typically used for [[api-gateway|caching]] frequently accessed data.
- **Consistency Models**: [[Memcached]] is eventually consistent since it doesn’t support transactions or locking mechanisms like [[Redis]].
- **Persistence Options**: Unlike [[Redis]], [[Memcached]] does not provide persistence options. Data stored in [[Memcached]] will be lost on a restart.

#### EXHAUSTIVE FEATURE LIST:
**[[Redis]]:**
- In-memory data store
- Support for complex data structures (strings, hashes, lists, sets, sorted sets)
- Transactions and Lua scripting support
- Persistence with RDB and AOF options
- Clustering with [[Redis Cluster]] or [[elasticache]]’s global replication feature
- Pub/Sub messaging

**[[Memcached]]:**
- Simple key-value cache
- Distributed [[api-gateway|caching]] using consistent hashing
- No support for transactions or complex data structures
- No persistence
- Limited query capabilities (get, set)

#### STEP-BY-STEP IMPLEMENTATION:
1. **Create [[elasticache]] Cluster**:
   - Use AWS Management Console or CLI to create a Redis or Memcached cluster.
2. **Configure [[appsync|Security]] and Networking**:
   - Set up [[security groups]] to control access.
   - Integrate with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] for network isolation.
3. **Parameter Group Configuration**:
   - Tune parameters like max memory limits, eviction [[policies]] (Redis only).
4. > [!warning] Quota
     Enable [[cloudwatch]] monitoring to track performance metrics.
     Set up alarms based on CPU utilization, memory usage, etc.

#### TECHNICAL LIMITS (2026):
- **[[Redis]]**: 
  - Maximum node size: Up to 1536 GB
  - Maximum number of nodes in a cluster: 20 per replication group
  - Hard limit on keys per node: Depends on memory and data types
- **[[Memcached]]**:
  - Maximum node size: Typically smaller than Redis, up to 75 GB
  - No built-in sharding; requires application-level handling

#### CLI & API GROUNDING:

```bash
# Create a [[Redis]] cluster
aws elasticache create-cache-cluster --cache-cluster-id my-redis-cluster \
    --engine redis --num-cache-nodes 2 --cache-node-type cache.r5.large \
    --security-group-ids sg-0123456789abcdef0 --subnet-group-name my-subnet-group

# Create a [[Memcached]] cluster
aws elasticache create-cache-cluster --cache-cluster-id my-memcached-cluster \
    --engine memcached --num-cache-nodes 2 --cache-node-type cache.r5.large \
    --security-group-ids sg-0123456789abcdef0 --subnet-group-name my-subnet-group
```

#### PRICING & TCO:
- **[[Redis]]**: 
  - Priced based on instance type and storage.
  - Additional costs for read replicas, global replication, etc.
- **[[Memcached]]**: 
  - Generally cheaper but less powerful in terms of data structures and operations.

**Optimization Strategies**:
- Use [[Redis]] when complex data manipulation is needed.
- Optimize memory usage by tuning eviction [[policies]].
- Leverage [[elasticache]]’s monitoring to scale resources dynamically.

#### COMPETITOR COMPARISON:
- **[[Redis]] vs. Memcached**: 
  - Redis offers more features (data structures, transactions) but at a higher cost and complexity.
  - [[Memcached]] is simpler, cheaper, but less flexible.

#### INTEGRATION:
- **[[cloudwatch]]**: Integration for monitoring cache performance metrics.
- **[[00_Master_Links_and_Intro|IAM]]**: Use [[00_Master_Links_and_Intro|IAM]] roles to control access to [[elasticache]] clusters.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Deploy clusters within VPCs for enhanced [[appsync|security]] and network isolation.

#### USE CASES:
- **[[Redis]]**:
  - Session management
  - [[api-gateway|Caching]] frequently accessed data (with complex queries)
  - Pub/Sub messaging patterns

- **[[Memcached]]**:
  - Simple [[api-gateway|caching]] of web pages or API responses
  - Load balancing across multiple instances for high availability

#### DECISION MATRIX: Features vs. Use Cases
| Feature               | Redis                | Memcached             |
|-----------------------|----------------------|-----------------------|
| Data Structures       | Complex (strings, hashes, etc.) | Simple key-value pairs     |
| Consistency Models    | Strong for single-key operations | Eventual consistency      |
| Persistence           | RDB/AOF              | None                  |
| Use Cases             | Session management, [[api-gateway|caching]] with complex queries | Simple web page/API response [[api-gateway|caching]] |

#### 2026 UPDATES:
- **[[Redis]]**: Enhanced support for multi-AZ deployments.
- **[[Memcached]]**: Improved performance and reliability.

#### EXAM SCENARIOS:
- Questions on configuring Redis/AOF persistence.
- Scenarios involving cache eviction [[policies]] and their impact on application performance.

#### INTERACTION PATTERNS:
- **Service-to-service interactions**:
  - [[Redis]] nodes communicating in a cluster for sharding, replication.
  - Memcached instances using consistent hashing to distribute keys across multiple nodes.

#### STRATEGIC ALIGNMENT:
- Focus on understanding the differences between Redis and Memcached in terms of features and use cases.
- Emphasize practical application scenarios relevant to enterprise environments.

> [!abstract] Exam Tip
Focus on high-weight topics like data consistency models and integration with [[00_Master_Links_and_Intro|other AWS services]] for exam preparation.

#### EXAM AUDIT:

##### Distractor Analysis:
1. **Scenarios Where This Service is a "Wrong" Answer:**
   - **Scenario A:** When you require transactional consistency or complex data structures beyond key-value pairs, such as sets, lists, hashes, and sorted sets.
     - *Explanation:* Redis supports rich data types like these, while Memcached only handles simple key-value operations.

   - **Scenario B:** In cases where persistence is a necessity (e.g., you need to prevent data loss during node failures).
     - *Explanation:* Redis offers several storage options including RDB and AOF persistence. Memcached does not support any form of persistence natively.

   - **Scenario C:** When high availability and automatic failover are required.
     - *Explanation:* Redis provides enhanced replication, automatic failover, and read replicas for improved reliability. Memcached lacks built-in high-availability features.

2. > [!danger] Distractor
     Performance vs Cost Trade-offs:
     - **Redis**:
       - *Pros*: Offers better performance with advanced data structures, supports persistence, and provides enhanced replication capabilities.
       - *Cons*: Generally more expensive than Memcached due to the additional features and higher compute requirements.
     - **Memcached**:
       - *Pros*: Simpler architecture leading to lower costs and easier setup. Excellent for simple [[api-gateway|caching]] needs where speed is paramount.
       - *Cons*: Lacks advanced data structures, persistence options, and high-availability features.

3. **Enterprise Governance:**
   - **[[organizations|AWS Organizations]]:** Use [[organizations|AWS Organizations]] to manage multiple accounts under a single entity, ensuring that all [[elasticache]] instances are governed by consistent [[policies]] across the organization.
   - **Service Control [[policies]] (SCPs):** Implement SCPs to enforce compliance with organizational standards and restrict access to sensitive resources like [[elasticache]] clusters.
   - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Utilize [[ram]] to share specific [[elasticache]] Redis clusters across multiple accounts within your [[AWS Organization]], streamlining resource management and reducing costs.
   - **[[00_Master_Links_and_Intro|CloudTrail]]:** Enable [[00_Master_Links_and_Intro|CloudTrail]] to log all API calls made against [[elasticache]]. This helps in auditing access patterns, troubleshooting issues, and maintaining compliance.

4. > [!warning] Quota
     Complex Failure Modes:
     - *[[kms]] Key Policy Conflicts*: When using [[00_Master_Links_and_Intro|AWS KMS]] for encryption, ensure that the key [[policies]] are correctly configured to allow both [[elasticache]] nodes and [[00_Master_Links_and_Intro|IAM]] roles to access them. Misconfigured [[policies]] can lead to [[api-gateway|authentication]] failures or unauthorized access.
     - *Replication Lag*: Monitor replication lag in Redis clusters closely as high latency can indicate network issues or excessive write loads. Use [[cloudwatch]] metrics to identify and troubleshoot these performance bottlenecks.

5. **Decision Matrix:**
   | Use Case                | Solution Recommendation          |
   |-------------------------|----------------------------------|
   | Simple Key-Value [[api-gateway|Caching]]| Memcached                       |
   | Advanced Data Structures| Redis                           |
   | High Availability       | Redis (with automatic failover)  |
   | Persistence Requirements| Redis                           |

6. **Recent Updates:**
   - **General Availability Features for 2026:** Anticipate features such as enhanced encryption options, improved monitoring and alerting capabilities, and support for newer Redis versions.
   - **Deprecated Items for 2026:** [[nonportable_instructions|Review]] AWS documentation for any deprecated features like specific [[elasticache]] engine versions or legacy APIs that may no longer be supported.

By carefully considering these factors, you can make informed decisions about when to use Amazon [[elasticache]] with [[Redis]] versus Memcached based on your enterprise needs and governance requirements.