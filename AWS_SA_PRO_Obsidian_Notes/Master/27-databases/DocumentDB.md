## Advanced Architecture
At its core, Amazon DocumentDB is a fully managed document database service that supports MongoDB workloads. It offers scalable performance, durability, and availability by automatically spreading data across multiple isolated systems. The storage layer uses a variant of B-trees designed for SSDs, while the compute layer has a shared-nothing architecture for high request throughput. DocumentDB also supports multi-region replication through Global Database Clusters. This allows for low-latency reads and writes across geographic regions.

## Comparison & Anti-Patterns

| Service             | Ideal Usage                                                              | Anti-Patterns                                                          |
|---------------------|--------------------------------------------------------------------------|-------------------------------------------------------------------------|
| DocumentDB          | High-performance, MongoDB-compatible, document-based workloads         | Direct manipulation of underlying storage engine or MongoDB features not supported by DocumentDB |
| [[dynamodb]]           | Fully managed NoSQL key-value store and document model                | Workloads requiring MongoDB compatibility or complex transactions      |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (MySQL, PostgreSQL)   | Managed relational databases                                            | Non-relational data models or unsupported engines                     |
| Keyspace (Cassandra)    | Managed Cassandra clusters                                               | Non-Cassandra workloads                                                |
| [[Collab_Notes_detailed/Databases/Neptune|Neptune]] (Graph DB)   | Graph databases using Gremlin, SPARQL, or OpenCypher query languages    | Non-graph workloads                                                   |

## [[appsync|Security]] & Governance
DocumentDB provides several options for securing your data. These include network isolation via [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|security]] groups, encryption at rest using AWS Key Management Service ([[kms]]), and encryption in transit using Transport Layer [[appsync|Security]] (TLS). To control access, you can create fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] based on specific API operations. Here's an example JSON policy snippet allowing `db:createCollection` action:
```json
{
    "Effect": "Allow",
    "Action": ["docdb:CreateCollection"],
    "Resource": "*"
}
```
Cross-account access is possible using resource-based [[policies]] attached directly to DocumentDB resources. For centralized governance, you can enforce Organization Service Control [[policies]] (SCPs) restricting DocumentDB usage within an [[AWS Organization]].

## Performance & Reliability
To optimize performance, DocumentDB offers throttling limits based on provisioned capacity. In case of exceeding these limits, applying exponential backoff strategies helps manage transient [[api-gateway|errors]]. Implementing horizontal scaling via read replicas improves read performance and enhances [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls in DocumentDB involve choosing appropriate instance types, setting up auto scaling, and utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|reserved instances]] when necessary. Calculating costs requires understanding the pricing structure, which includes hourly rates for instances, data transfer fees, and optional backup storage charges.

## Professional Exam Scenarios
### Scenario 1: Multi-Account Strategy
Your company runs multiple projects across various AWS accounts but wants to share a single DocumentDB cluster among them. How would you implement cross-account access and secure data?

**Correct Answer:**
Implement cross-account access by creating a resource-based policy on the DocumentDB cluster granting permissions to the desired [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or users from other accounts. Secure data using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|security]] groups, [[kms]] keys, and TLS encryption.

**Incorrect Answers:**

* Sharing master user credentials: This poses a significant [[appsync|security]] risk as it grants full access to the DocumentDB cluster.
* Using [[sns]] topics to distribute write permissions: [[sns]] is not designed for managing database permissions.

### Scenario 2: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] Pattern
A mission-critical application requires minimal downtime during maintenance or failure scenarios. Design a highly available and durable solution using DocumentDB.

**Correct Answer:**
Use DocumentDB Global Database Clusters for multi-region replication and automatic failover. Implement read replicas for improved read performance and additional [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities. Ensure proper backups and recovery point objectives (RPOs) and recovery time objectives (RTOs) are met.

**Incorrect Answers:**

* Utilizing only single-region DocumentDB clusters without read replicas: This setup lacks redundancy and may result in higher downtimes during failures.
* Employing manual synchronization between separate DocumentDB clusters: This method introduces unnecessary complexity and potential human [[api-gateway|errors]].