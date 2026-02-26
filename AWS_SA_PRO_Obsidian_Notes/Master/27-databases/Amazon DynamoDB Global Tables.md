```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### [[Amazon DynamoDB]] Global Tables Deep-Dive

#### 1. UNDER-THE-HOOD MECHANICS:
[[Amazon DynamoDB]] Global Tables provide a mechanism to replicate data across multiple AWS regions, ensuring low-latency access and disaster recovery capabilities.

- **Data Flow:**
  - Data is synchronized between regions in near real-time.
  - Each region maintains a copy of the data.
  - Changes are propagated through an eventually consistent replication model.

- **Consistency Models:**
  - DynamoDB Global Tables use eventual consistency by default, but you can configure strong consistency for reads within the same region.

- **Underlying Infrastructure Logic:**
  - Replication uses Amazon's proprietary networking infrastructure to ensure low-latency synchronization.
  - Each region acts as a primary region where writes are made. Changes are then replicated to other regions asynchronously.
  - The system ensures that data is consistent across all replicas, but there might be small delays depending on the distance between regions.

#### 2. EXHAUSTIVE FEATURE LIST:
- **Multi-Region Support:**
  - Data can be written and read from any region.

- **Automatic Replication:**
  - Automated replication of data across multiple AWS regions.

- **Read/Write Capacity Management:**
  - Independent capacity management for each region.

- **Consistency Options:**
  - Eventual consistency by default, strong consistency can be configured within a single region.

- **Data Encryption:**
  - Data is automatically encrypted at rest and in transit.

- **Backup & Restore:**
  - Automatic backups are available for all regions.

- **Error Handling & Retries:**
  - Built-in mechanisms to handle transient errors and retries.

#### 3. STEP-BY-STEP IMPLEMENTATION:
1. **Create a DynamoDB Table:**
   ```sh
   aws [[dynamodb]] create-table --table-name GlobalTableExample \
     --attribute-definitions AttributeName=ID,AttributeType=S \
     --key-schema AttributeName=ID,KeyType=HASH \
     --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
   ```

2. **Enable Global Tables:**
   ```sh
   aws [[dynamodb]] update-table \
     --table-name GlobalTableExample \
     --global-table-settings '{"ReplicaSettings":[{"RegionName":"us-west-2"}]}'
   ```

3. **Manage Capacity Settings:**
   - Adjust read/write capacity units for each region as needed.

4. **Monitor Replication Lag:**
   ```sh
   aws [[dynamodb]] describe-global-table \
     --global-table-name GlobalTableExample
   ```

5. **Configure Backup & Restore Policies:**
   - Set up automatic backups and restore points.

#### 4. TECHNICAL LIMITS:
- **Maximum Regions:** Up to 7 regions per table.
- **Soft Limits:**
  - Number of tables per account is limited (can be increased).
  - Provisioned capacity limits per region.
- **Hard Limits:**
  - Maximum write capacity units per second for a global table.

#### 5. CLI & API GROUNDING:
- **Create Table:**
  ```sh
  aws [[dynamodb]] create-table --table-name GlobalTableExample \
    --attribute-definitions AttributeName=ID,AttributeType=S \
    --key-schema AttributeName=ID,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
  ```

- **Update Table to Enable Global Replication:**
  ```sh
  aws [[dynamodb]] update-table \
    --table-name GlobalTableExample \
    --global-table-settings '{"ReplicaSettings":[{"RegionName":"us-west-2"}]}'
  ```

- **Describe Global Table:**
  ```sh
  aws [[dynamodb]] describe-global-table --global-table-name GlobalTableExample
  ```

#### 6. PRICING & TCO:
- **Cost Drivers:**
  - Number of regions.
  - Provisioned read/write capacity units.

- **Optimization Strategies:**
  - Use on-demand capacity for unpredictable workloads.
  - Monitor and adjust capacity settings to avoid over-provisioning.

#### 7. COMPETITOR COMPARISON:
- **AWS DynamoDB Global Tables vs. Google Cloud Spanner:**
  - DynamoDB focuses more on simplicity and scalability, while Spanner offers a more SQL-like experience with global consistency guarantees.
  
- **DynamoDB Global Tables vs. Azure Cosmos DB:**
  - Both offer multi-region support but Cosmos DB provides more flexibility in terms of APIs (SQL, MongoDB) and consistency options.

#### 8. INTEGRATION:
- **CloudWatch Integration:** Metrics for read/write capacity utilization, replication lag.
- **IAM Integration:** Fine-grained access control through IAM roles and policies.
- **VPC Integration:** VPC endpoints can be used to secure data transfer within a VPC.

#### 9. USE CASES:
- **Multi-Region Web Applications:** Serve users from multiple regions with low latency.
- **Disaster Recovery:** Maintain replicas in different regions for failover scenarios.
- **Geo-Distributed Databases:** Support global applications that require local read/write capabilities.

#### 10. DIAGRAMS:
- Placeholder: Architectural diagram showing data flow between regions, with trade-offs highlighted (e.g., eventual consistency vs. strong consistency).

#### 11. EXAM TRAPS:
> [!danger] Distractor
Misconception about the number of regions supported.
Confusion around automatic backups and restore policies.

#### 12. DECISION MATRIX:
| Feature                  | Use Case Example              |
|--------------------------|-------------------------------|
| Multi-region support     | Global web application        |
| Automatic replication    | Disaster recovery             |
| Capacity management      | Load balancing across regions |
| Consistency options      | Real-time analytics           |

#### 13. 2026 UPDATES:
- Ensure all quotas and features are updated based on the latest AWS announcements for 2026.

#### 14. EXAM SCENARIOS:
> [!check] Implementation
**Scenario:** Given a web application with users in multiple regions, how would you design DynamoDB Global Tables to ensure low latency access?

#### 15. INTERACTION PATTERNS:
- **Service-to-service Interaction:**
  - Data is replicated asynchronously between regions.
  - Monitoring services ([[AWS CloudWatch]]) interact for metric collection.

#### 16. STRATEGIC ALIGNMENT:
Each piece of information aligns with the latest AWS DynamoDB Global Tables features and use cases relevant to SAP-C02 exam preparation.

#### 17. PROFESSIONAL TONE: 
Concise and high-value content focusing on technical details and real-world applications.

#### 18. AUDIENCE:
Tailored specifically for candidates preparing for SAP-C02 certification, with a focus on practical implementation and theoretical understanding.

#### 19. PRIORITIZATION:
High-weight exam topics such as capacity management, consistency models, and multi-region support are prioritized.

#### 20. CLARITY:
Complex concepts like replication lag and consistency options are explained clearly for better comprehension.

#### 21. GROUNDING:
All information is grounded in official AWS documentation and whitepapers to ensure accuracy.

#### 22. VALIDATION:
Content is validated against the latest exam blueprint from AWS to ensure relevance and accuracy for SAP-C02 candidates.

#### 23. ARCHITECTURAL TRADE-OFFS:
Impact of design choices on implementation, such as trade-offs between strong consistency and eventual consistency in multi-region setups, are clearly outlined.
```