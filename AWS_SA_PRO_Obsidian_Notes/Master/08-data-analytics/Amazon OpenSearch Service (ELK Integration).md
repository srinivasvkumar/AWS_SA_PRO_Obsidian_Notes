```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Amazon OpenSearch Service (formerly [[Elasticsearch Service]])

#### UNDER-THE-HOOD MECHANICS:
- **Data Flow:** Data is ingested through various methods such as direct indexing, bulk API calls, or via [[Kinesis Data Streams]]. The data is then distributed across shards for high availability and performance.
- **Consistency Models:** Supports eventual consistency by default but can be configured to use near real-time (NRT) refresh intervals for more immediate visibility of indexed documents.
- **Underlying Infrastructure Logic:** Uses a cluster management system to handle scaling, node health checks, and recovery. Each shard is replicated across multiple nodes to ensure fault tolerance.

#### EXHAUSTIVE FEATURE LIST:
1. **Multi-AZ Clusters** - Provides high availability by replicating data across zones within the same region.
2. **Cross-Cluster Replication (CCR)** - Enables near real-time replication of data from one OpenSearch cluster to another, often used for disaster recovery or regional data distribution.
3. **Auto Scaling Policies** - Automatically adjusts the number of instances based on CPU utilization and other metrics.
4. **Machine Learning Integrations** - Offers machine learning capabilities for anomaly detection, forecasting, and classification using [[Amazon SageMaker]].
5. **Security Enhancements:** Kibana authentication via IAM roles, fine-grained access control (FGAC) for managing security at the document level.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set up an OpenSearch cluster:**
   - Choose instance types and node counts.
   - Configure VPC settings to ensure proper network isolation.
2. **Ingest Data:**
   - Use [[Kinesis Data Streams]] or direct API calls for data ingestion.
3. **Configure Index Management:**
   - Define indices with appropriate mappings and shard allocations.
4. **Set up Monitoring and Alerts:**
   - Integrate CloudWatch for monitoring performance metrics.

#### TECHNICAL LIMITS (2026):
- **Soft Limits:** Number of clusters per account varies; default is 10 but can be increased by contacting AWS Support.
- **Hard Limits:** Maximum number of shards per cluster is typically 1,000.

#### CLI & API GROUNDING:
- **Create a Domain:**
  ```bash
  [[00_Master_Links_and_Intro|aws opensearch]] create-domain --domain-name my-opensearch-cluster --engine-version OpenSearch_2.4 --cluster-config InstanceType=r5.large.search,InstanceCount=3 --ebs-options EBSEnabled=true,VolumeType=gp2,VolumeSize=10
  ```
- **Add a Data Policy:**
  ```bash
  [[00_Master_Links_and_Intro|aws opensearch]] put-data-policy --policy-name MyDataPolicy --policy-document file://data-policy.json
  ```

#### PRICING & TCO:
- **Cost Drivers:** Instance types (memory and CPU), storage type, number of nodes, and data transfer costs.
- **Optimization Strategies:** Use On-Demand Instances for predictable workloads; apply Reserved Instances or Savings Plans for cost savings over long-term usage.

#### COMPETITOR COMPARISON:
- **Elastic Cloud (Elasticsearch Service):** Offers a more flexible pricing model with dedicated hardware options but lacks some of the AWS integration benefits.
- **Azure Cognitive Search:** Provides integrated AI capabilities out-of-the-box, but has less mature security features compared to OpenSearch.

#### INTEGRATION:
- **CloudWatch:** Metrics for CPU utilization, JVM heap usage, and index health are automatically sent to [[cloudwatch]].
- **IAM:** Uses IAM roles for authentication in Kibana and supports FGAC within the cluster itself.
- **VPC:** Can be deployed within a VPC for enhanced security.

#### USE CASES:
1. **Log Aggregation:** Centralize logs from various sources into one searchable index.
2. **Real-Time Analytics:** Use machine learning to detect anomalies in application performance or user behavior.

#### DIAGRAMS:
- Placeholder: High-level architecture diagram showing OpenSearch cluster, Kibana, and data ingestion paths via Kinesis Data Streams.

> [!abstract] Exam Tip
> Confusing Index Sharding with Partitioning:
Understanding the difference between sharding for scalability and partitioning in databases.
>
> Misunderstanding Multi-AZ Clusters:
Ensure that a Multi-AZ cluster means high availability within a region, not across regions.

#### DECISION MATRIX:

| Feature                | Use Case: Log Aggregation        | Use Case: Real-Time Analytics  |
|------------------------|----------------------------------|-------------------------------|
| Index Sharding          | Essential for distributing logs  | Essential for scaling analytics|
| Cross-Cluster Replication| Useful for disaster recovery    | Not typically needed           |
| Machine Learning        | Optional                         | Highly beneficial             |

#### 2026 UPDATES:
- **New Engine Versions:** OpenSearch 3.x will offer enhanced security features and better performance.
- **Enhanced Security Options:** FGAC will be more widely adopted due to its granular control.

#### EXAM SCENARIOS:
1. **Scenario:** Determining the optimal node count for a highly available log aggregation cluster.
2. **Scenario:** Configuring an OpenSearch domain with Kibana and IAM authentication.

> [!warning] Quota
> Keep track of AWS service quotas such as the number of clusters per account, which defaults to 10 but can be increased by contacting support.

#### INTERACTION PATTERNS:
- **Service-to-service Interaction:** OpenSearch interacts with CloudWatch for monitoring, IAM for security, and VPC for network isolation.

#### STRATEGIC ALIGNMENT:
Each piece of information aligns with high-weight exam topics such as cluster management, security configurations, and cost optimization strategies.

> [!danger] Distractor
> AWS Kinesis might be more appropriate than OpenSearch for real-time streaming data analysis that requires ultra-low latency.

```

### AI Linked Diagram
![[Screenshot 2026-01-03 at 10.23.00 PM.png]]
