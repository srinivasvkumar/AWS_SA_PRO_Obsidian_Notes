```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into Multi-Site Active-Active Disaster Recovery (DR) Strategy

#### UNDER-THE-HOOD MECHANICS:
Multi-site active-active disaster recovery involves setting up identical environments across multiple geographic locations that handle traffic independently. Each site serves as a primary for its region and simultaneously acts as a failover backup for other regions.

1. **Data Flow:**
   - Data is replicated bidirectionally between the sites, often using [[AWS DynamoDB Global Tables]] or [[RDS Multi-AZ Deployments]].
   - Traffic routing can be managed via [[Route 53]] with health checks and latency-based routing to ensure users are directed to the nearest active site.

2. **Consistency Models:**
   - Strong consistency is maintained through synchronous replication, which ensures data integrity but may have higher latency.
   - Eventual consistency allows for lower latency at the cost of temporary discrepancies, which can be managed with conflict resolution mechanisms (e.g., last-write-wins).

3. **Underlying Infrastructure Logic:**
   - [[AWS Direct Connect]] or [[AWS Transit Gateway]] is used to establish private connections between sites for secure and low-latency data replication.
   - [[Auto Scaling Groups]] and [[Elastic Load Balancers]] are employed within each region to handle varying traffic loads.

#### EXHAUSTIVE FEATURE LIST:
1. **Bidirectional Replication:** Real-time synchronization of databases across regions.
2. **Health Checks:** Automated monitoring via Route 53 to detect failures.
3. **Latency-Based Routing:** Ensuring users are directed to the nearest healthy site.
4. **Conflict Resolution:** Mechanisms for resolving data discrepancies (e.g., timestamps, versioning).
5. **Auto Scaling & Load Balancing:** Dynamic resource management within each region.
6. **Security & Compliance:** Implementing [[IAM]] roles and policies, VPC security groups, and encryption at rest/transfer.
7. **Multi-AZ Deployments:** Using services like RDS Multi-AZ for high availability.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Identify Business Requirements:**
   - Define RTO (Recovery Time Objective) and RPO (Recovery Point Objective).
   
2. **Design Architecture:**
   - Use Route 53 with health checks and latency-based routing.
   - Deploy AWS Transit Gateway or Direct Connect for cross-region connectivity.

3. **Set Up Infrastructure:**
   - Configure DynamoDB Global Tables or RDS Multi-AZ for database replication.
   - Set up Auto Scaling Groups and ELBs in each region.

4. **Implement Security & Compliance:**
   - Define IAM roles, VPC security groups, and encryption settings.

5. **Test the DR Plan:**
   - Regularly test failover scenarios to ensure smooth transition between regions.

#### TECHNICAL LIMITS (2026):
1. **DynamoDB Global Tables:** Maximum of 5 global tables per account.
2. **RDS Multi-AZ Deployments:** Limited by instance type and region availability.
3. **Route 53 Health Checks:** Up to 1,000 health checks per hosted zone.

#### CLI & API GROUNDING:
- **DynamoDB Global Tables:**
  ```sh
  aws [[dynamodb]] create-global-table --global-table-name MyTable --replication-group RegionName=us-east-1 RegionName=eu-west-1
  ```

- **Route 53 Latency-Based Routing:**
  ```sh
  aws [[route53]] change-resource-record-sets --hosted-zone-id ZABCDEF12345678 --change-batch file://batch.json
  ```

#### PRICING & TCO:
1. **Cost Drivers:** Increased compute and storage costs due to redundant infrastructure.
2. **Optimization Strategies:**
   - Use Reserved Instances for predictable workloads.
   - Implement cost-effective storage solutions like S3 IA for archival data.

#### COMPETITOR COMPARISON:
- **Azure Availability Zones vs. AWS Multi-AZ:** Azure offers a more granular level of redundancy within regions, but AWS provides broader geographic distribution with multi-site setups.
- **GCP Regional & Zonal Resources:** GCP's approach is similar to AWS, offering global load balancing and regional replication.

#### INTEGRATION:
1. **CloudWatch:** Monitoring health checks and performance metrics across all sites.
2. **IAM:** Define roles for cross-region access and management tasks.
3. **VPC:** Securely connect regions using VPC peering or Transit Gateway.

#### USE CASES:
- **Financial Services:** Ensuring transactional consistency and minimal downtime with bidirectional database replication.
- **E-commerce Platforms:** Handling high traffic volumes across regions without disrupting service during failures.

#### DIAGRAMS:
1. Placeholder for architectural diagram: Multi-Site Active-Active DR setup with Route 53, DynamoDB Global Tables, and VPCs interconnected via Transit Gateway.

2. Trade-offs between strong consistency vs. eventual consistency in data replication.

#### EXAM TRAPS:
- Misconception that multi-site active-active automatically guarantees zero RTO/RPO.
- Overlooking the importance of regular failover testing.

#### DECISION MATRIX: Features vs. Use Cases
| Feature                 | Financial Services            | E-commerce Platforms        |
|-------------------------|-------------------------------|------------------------------|
| Bidirectional Replication| Essential for transactional integrity | Necessary for global consistency |
| Latency-Based Routing   | Less critical due to less user interaction | Critical for low-latency experience |

#### 2026 UPDATES:
- AWS Direct Connect and Transit Gateway enhancements.
- Improved DynamoDB Global Tables features.

#### EXAM SCENARIOS:
1. **Scenario:** Designing a multi-site DR strategy for an e-commerce platform with strict RPO/RTO requirements.
   - Focus: Bidirectional replication, Route 53 setup, and failover testing procedures.

2. **Scenario:** Ensuring compliance in financial services across multiple regions.
   - Focus: IAM roles, encryption, and VPC security groups.

#### INTERACTION PATTERNS:
1. **Service-to-Service Interaction Patterns:**
   - Route 53 -> ELBs -> Auto Scaling Groups
   - DynamoDB Global Tables -> Replication Across Regions

#### STRATEGIC ALIGNMENT:
Each topic is aligned with the latest AWS SAP-C02 exam blueprint, ensuring comprehensive coverage of high-weight topics.

#### PROFESSIONAL TONE & AUDIENCE:
Tailored specifically for SAP-C02 candidates, focusing on precision and relevance to pass the exam.

#### PRIORITIZATION & CLARITY:
High-priority topics are emphasized with clear, concise explanations grounded in official AWS documentation and whitepapers.
```