**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[dynamodb|DynamoDB Global Tables]] provide a multi-region, multi-master replication feature. It allows low-latency reads and writes across multiple regions. Under the hood, [[dynamodb]] uses a Paxos algorithm for strong consistency and data versioning. Data is stored in SSD-backed nodes, and each node handles a subset of the data. The Paxos algorithm ensures that all nodes agree on the order of writes, providing strong consistency.

Global Tables automatically handle failover, and you can perform read and write operations even when some regions become unavailable. [[dynamodb]] also supports Point-in-Time Recovery (PITR) for Global Tables, which enables backup and restore capabilities at the global level.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
| --- | --- |
| [[dynamodb|DynamoDB Global Tables]] | Multi-region applications requiring low latency and high availability. |
| [[dynamodb|DynamoDB Accelerator (DAX)]] | Improving read performance for specific workloads with [[api-gateway|caching]]. |
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Replication | Homogeneous database engine replication, not suitable for NoSQL databases. |

*Anti-pattern*: Using [[dynamodb|DynamoDB Global Tables]] for multi-region deployments without considering the additional costs and potential throttling issues due to increased write capacity unit (WCU) requirements.

**[[RDS_Instance_Types|3. Security & Governance]]**

To manage cross-account access, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role that grants permissions to other accounts using resource-based [[policies]]. For example, here's how you might set up a policy granting `dbsa-external` access to your [[dynamodb]] table in `dbsa-main` account:

```json
{
  "Version": "2012-10-17",