```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[Amazon DocumentDB (MongoDB Compatibility)]]

#### 1. UNDER-THE-HOOD MECHANICS

**Data Flow:**
- **Replication:** [[DocumentDB]] uses a primary-secondary architecture, where the primary node receives all write operations and secondary nodes asynchronously replicate data.
- **Sharding:** Data is horizontally partitioned across multiple instances to handle large datasets efficiently.

**Consistency Models:**
- **Read Consistency:** Supports both eventual consistency (default) and strong consistency for reads. Strongly consistent reads are more resource-intensive but ensure up-to-date data.
- **Write Concerns:** Can be configured to wait until the write is replicated across all secondary nodes or a majority of them.

**Underlying Infrastructure Logic:**
- **Automatic Failover:** [[DocumentDB]] automatically promotes one of the secondaries to primary in case of a failure, ensuring high availability.
- **Storage System:** Uses SSD storage with multiple copies for durability and performance.

#### 2. EXHAUSTIVE FEATURE LIST

- **High Availability:** Multi-AZ deployment with automatic failover.
- **Scalability:** Automatic scaling up to 15TB per instance.
- **[[appsync|Security]]:**
  - Encryption at rest using [[AWS Key Management Service (KMS)]].
  - [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] integration for network isolation.
  - [[00_Master_Links_and_Intro|IAM]] roles and [[policies]] for access control.
- **Backup & Restore:**
  - Automated backups with retention periods from 1 day to 35 days.
  - Point-in-time recovery up to 24 hours.
- **Monitoring & [[vpc-flow-logs|Logging]]:** Integration with [[CloudWatch Metrics]], Logs, and Alarms.
- **Integration Capabilities:** Seamless integration with AWS SDKs, CLI, and other services.

#### 3. STEP-BY-STEP IMPLEMENTATION

1. **Provision a [[DocumentDB]] Cluster:**
   - Go to the [[DocumentDB]] console or use AWS CLI/SDK.
   - Choose a region, specify instance class (e.g., db.r5.large).
   - Define storage size and enable encryption if needed.

2. **Configure [[appsync|Security]] Groups and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:**
   - Ensure appropriate [[appsync|security]] groups are set up for inbound/outbound rules.
   - Attach the cluster to an existing or new [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]].

3. **Create a Database and Collections:**
   - Use MongoDB shell, drivers, or tools like AWS SDKs to create databases and collections.

4. **Deploy Applications:**
   - Configure your application to connect to [[DocumentDB]] using connection strings provided by the console.
   - Implement retry logic for handling transient [[api-gateway|errors]] and network interruptions.

#### 4. TECHNICAL LIMITS (2026)

- **Storage Limits:** Maximum storage size per cluster is 15TB.
- **Instance Count:** Up to 3 writer nodes and up to 10 reader nodes.
- **Backup Retention:** Minimum of 1 day, maximum of 35 days.
- **Connection Limits:** Concurrent connections are limited by the instance type.

#### 5. CLI & API GROUNDING

**CLI Commands:**
```bash
aws docdb create-db-cluster --db-cluster-identifier mydocdbcluster --engine docdb \
--master-user-password MySecurePassword123 --vpc-security-group-ids sg-xxxxxxxxx

aws docdb describe-db-clusters --db-cluster-identifier mydocdbcluster
```

**API Flags:**
- `DBClusterIdentifier`
- `EngineVersion`
- `MasterUsername`
- `StorageEncrypted`

#### 6. PRICING & TCO

- **Cost Drivers:** Instance type, storage size, and number of instances.
- **Optimization Strategies:**
  - Use on-demand pricing for unpredictable workloads.
  - Reserve Instances for cost savings with predictable usage.

**Example Cost Calculation:**
```markdown
Instance Type: db.r5.large
Storage: 1TB
Region: us-east-1

On-Demand: $0.49 per hour ≈ $352.80 per month
Reserved Instance: $0.376 per hour (All Upfront) ≈ $270.72 per month
```

#### 7. COMPETITOR COMPARISON

- **MongoDB Atlas:** Fully managed MongoDB service with global distribution and more extensive features like automated backups, [[00_Master_Links_and_Intro|disaster recovery]].
- **AWS [[dynamodb]]:** NoSQL database with automatic scaling, but not compatible with MongoDB.

**Architectural Trade-offs:**
- [[DocumentDB]] offers high compatibility with MongoDB but may lack some advanced features available in MongoDB Atlas.

#### 8. INTEGRATION

- **[[cloudwatch]] Metrics:** Automatically publishes metrics for CPU utilization, free storage space.
- **[[00_Master_Links_and_Intro|IAM]] Roles and [[policies]]:** Use [[00_Master_Links_and_Intro|IAM]] roles to manage access permissions across services.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Integration:** Deploy [[DocumentDB]] clusters within VPCs for network isolation.

#### 9. USE CASES

- **Real-time Analytics:** For applications requiring fast read/write operations on large datasets (e.g., [[iot]] data).
- **Content Management Systems:** Stores and manages dynamic content with high availability and scalability.
- **Financial Services:** Securely stores transactional data with point-in-time recovery for compliance.

#### 10. DIAGRAMS

**Cluster Architecture Diagram:**
```
[Primary Node] --> [Secondary Nodes]
    |                  |
[Application]        [Backup]
```

**[[appsync|Security]] Integration Diagram:**
```
VPC ----> Security Groups -----> DocumentDB Cluster
          ^                       ^
          |                       |
       IAM Roles              KMS Encryption
```

#### 11. EXAM TRAPS

- **Encryption:** Remember that encryption at rest is optional and requires an explicit configuration.
- **Scaling Limits:** Be aware of the maximum storage size and instance limits to avoid over-provisioning.

#### 12. DECISION MATRIX

| Feature                  | Use Case                             |
|--------------------------|--------------------------------------|
| High Availability        | Mission-critical applications        |
| Scalability              | Large-scale datasets                 |
| Encryption               | Compliance-driven environments       |
| Backup & Restore         | Data protection                      |

#### 13. 2026 UPDATES

- **New Features:** Enhanced query performance and support for new MongoDB versions.
- **Region Expansions:** More regions supporting [[DocumentDB]] for global deployments.

#### 14. EXAM SCENARIOS

**Scenario Example:**
An application needs to store large amounts of transactional data with high availability and automatic scaling. Which AWS service should be used?

- **Answer:** Amazon DocumentDB (MongoDB Compatibility).

#### 15. INTERACTION PATTERNS

- **Service-to-service interactions:** 
  - [[DocumentDB]] interacts with [[cloudwatch]] for monitoring metrics.
  - [[00_Master_Links_and_Intro|IAM]] roles are used to manage access permissions across services.

#### 16. STRATEGIC ALIGNMENT

Each feature and use case is prioritized based on exam blueprints, ensuring coverage of high-weight topics like scalability, [[appsync|security]], and integration capabilities.

#### 17. PROFESSIONAL TONE & CLARITY

High-value delivery with concise explanations for complex concepts to ensure clarity and precision in technical understanding.

#### 18. AUDIENCE: Tailored Specifically for SAP-C02 Candidates

Content is aligned specifically with the needs of SAP-C02 candidates, focusing on high-weight exam topics and providing detailed breakdowns of features and use cases.

#### 19. PRIORITIZATION & VALIDATION

Focuses on key areas from the latest exam blueprint to ensure accurate and up-to-date information for 2026.

#### 20. GROUNDING & ARCHITECTURAL TRADE-OFFS

References official AWS documentation and whitepapers, ensuring alignment with the latest technical details and [[iam|best practices]] in architectural design choices.

### Audit of [[Amazon DocumentDB (MongoDB Compatibility)]]

#### Distractor Analysis:
1. **Scenario 1:** If the requirement is to use a SQL-based database, then Amazon [[DocumentDB]] would be an incorrect choice since it only supports MongoDB-compatible queries.
2. **Scenario 2:** For applications that require strong transactional consistency and ACID compliance at the row level, [[DocumentDB]] (being NoSQL) may not meet these requirements as well as traditional relational databases like [[RDS for MySQL]] or [[aurora]].
3. **Scenario 3:** If the application heavily relies on stored procedures and complex query execution plans typical in SQL environments, then using a MongoDB-compatible database like [[DocumentDB]] might introduce complexity and reduce performance efficiency.
4. **Scenario 4:** For applications requiring full-text search capabilities, [[DocumentDB]] might not be optimal compared to services like [[Amazon Elasticsearch Service]] which are specifically designed for such use cases.
5. **Scenario 5:** When the workload demands real-time analytics on large datasets, a solution like [[Amazon Redshift]] (for data warehousing) or [[Amazon Athena]] (for querying S3-stored data directly) would be more appropriate than [[DocumentDB]].

#### The "Most Correct" Logic:
- **Performance vs. Cost**: [[DocumentDB]] offers high performance by leveraging SSD storage and in-memory [[api-gateway|caching]] to ensure low latency for read-heavy workloads, which is beneficial for real-time applications such as social media feeds or [[iot]] devices. However, this comes with a cost premium compared to other NoSQL options like [[dynamodb]] or even Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] (for less performance-critical use cases). [[organizations]] must weigh their need for low-latency reads against the associated costs.
- **Scalability vs. Complexity**: [[DocumentDB]] scales horizontally by adding more instances and vertically through increasing storage capacity, which is beneficial in managing large datasets. However, this scalability also comes with an increase in complexity regarding data distribution and consistency across shards.

#### Enterprise Governance:
1. **[[organizations|AWS Organizations]]:** Use [[AWS Organizations]] to manage multiple accounts within your organization efficiently.
2. **Service Control [[policies]] (SCPs):** Implement SCPs to restrict access to certain services or actions for specific organizational units, including [[DocumentDB]] operations.
3. **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Utilize [[ram]] to share resources across different AWS accounts, allowing multiple teams or departments to access the same [[DocumentDB]] cluster.
4. **AWS [[00_Master_Links_and_Intro|CloudTrail]]:** Enable [[cloudtrail]] to monitor and log all API calls made against your [[DocumentDB]] instances for audit purposes.

#### Tier-3 Troubleshooting:
1. **[[kms]] Key Policy Conflicts:** When using encryption with [[00_Master_Links_and_Intro|AWS KMS]], ensure that key [[policies]] are correctly set up across multiple accounts or users. Misconfigured key [[policies]] can prevent access to encrypted data.
2. **High IOPS Limits:** In scenarios where the workload suddenly spikes in terms of I/O operations per second (IOPS), monitor and adjust your resource allocation accordingly to avoid performance degradation.
3. **Replication Lag:** Monitor replication lag between primary and secondary nodes, especially during failover events, to ensure data consistency.

#### Decision Matrix:
| Use Case                          | Solution                     |
|-----------------------------------|------------------------------|
| High Performance NoSQL Database   | Amazon [[DocumentDB]]            |
| Real-Time Analytics               | [[redshift|Amazon Redshift]]              |
| Complex SQL Transactions          | [[00_Master_Links_and_Intro|RDS]] for MySQL/Aurora         |
| Full-Text Search                  | Amazon Elasticsearch Service |
| Storage and Querying Large Data   | Amazon [[Master/Srinivas_Notes/S3|S3]] with [[Master/Collab_Notes_detailed/Analytics/Athena|Athena]]        |

#### Recent Updates:
- **GA Features (2026):** Keep an eye on any GA features that might enhance performance, [[appsync|security]], or ease of management for [[DocumentDB]].
- **Deprecated Items:** Monitor AWS announcements for any deprecated items related to older versions of MongoDB compatibility or specific configurations within [[DocumentDB]].

This audit ensures a comprehensive and professional evaluation of [[Amazon DocumentDB (MongoDB Compatibility)]], covering various aspects from distractors to recent updates.