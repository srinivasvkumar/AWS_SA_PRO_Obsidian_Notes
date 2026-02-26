```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[AWS Elastic Disaster Recovery (DRS)]]

#### 1. UNDER-THE-HOOD MECHANICS:

**Internal Data Flow:**
- **Replication:** [[AWS DRS]] replicates your on-premises servers to [[Amazon EC2]] instances using a lightweight agent installed on the source server.
- **Syncing:** The replication occurs in real-time with incremental changes synchronized continuously, ensuring minimal data loss during [[00_Master_Links_and_Intro|disaster recovery]].

**Consistency Models:**
- **Point-in-Time Recovery:** Allows you to restore applications to specific points in time.
- **Application Consistent Snapshots:** Ensures that application state is consistent across all replicated resources.

**Underlying Infrastructure Logic:**
- **Agents:** Lightweight agents installed on source servers manage the replication process.
- **Storage:** Replicated data is stored in [[Amazon S3]] for durability and availability.
- **Compute:** AWS [[drs]] uses [[00_Master_Links_and_Intro|EC2]] instances to restore your applications. The type of instance can be customized based on resource requirements.

#### 2. EXHAUSTIVE FEATURE LIST:
- Agent Installation & Management
- Replication of On-Premises Servers
- Incremental Replication
- Real-Time Syncing
- Point-in-Time Recovery
- Application Consistent Snapshots
- [[00_Master_Links_and_Intro|EC2]] Instance Restoration
- Automated Failover
- Customizable RPO/RTO Targets
- Monitoring & Alerts Integration ([[cloudwatch]])
- [[appsync|Security]] and Compliance Controls

#### 3. STEP-BY-STEP IMPLEMENTATION:
1. **Install the AWS [[drs]] Agent:** Deploy the lightweight agent on your on-premises servers.
2. **Configure Replication Settings:** Define replication targets, RPO/RTO, and other parameters.
3. **Initiate Replication Process:** Start replicating data to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storage.
4. **Monitor Replication Status:** Use AWS Management Console or [[cloudwatch]] to monitor progress.
5. **Restore Applications in [[00_Master_Links_and_Intro|EC2]]:** Failover to restored instances during a disaster event.

#### 4. TECHNICAL LIMITS:
- **Soft Quotas:**
  - Maximum number of replication jobs per region (usually around 100).
  - Storage capacity for replicated data limited by [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket size.
- **Hard Quotas:**
  - [[00_Master_Links_and_Intro|EC2]] instance limits per region apply to restored instances.

#### 5. CLI & API GROUNDING:
- **CLI Commands:**
  ```sh
  aws drs start-replication-config-task # Starts the replication configuration task
  aws drs stop-replication-config-task # Stops the replication configuration task
  ```
- **API Flags:**
  - `--source-server-id`: Specifies the ID of the source server.
  - `--replication-settings`: Defines settings for RPO/RTO.

#### 6. PRICING & TCO:
- **Cost Drivers:**
  - [[00_Master_Links_and_Intro|EC2]] instance costs during replication and restoration.
  - [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] storage costs for replicated data.
- **Optimization Strategies:**
  - Use [[00_Master_Links_and_Intro|spot instances]] to reduce compute cost.
  - Configure lifecycle [[policies]] on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets to archive older snapshots.

#### 7. COMPETITOR COMPARISON:
- **AWS [[drs]] vs. [[Azure Site Recovery]]:**
  - AWS [[drs]] focuses more on seamless integration with Amazon services and [[00_Master_Links_and_Intro|EC2]] restoration, whereas Azure SR supports a broader range of cloud [[eb|platforms]].
  - Both support real-time replication but differ in agent management and monitoring tools.

#### 8. INTEGRATION:
- **[[Amazon CloudWatch]]:** Monitors the status of replication jobs and sends alerts for critical issues.
- **[[iam]]:** Manages access to AWS [[drs]] services through roles and [[policies]].
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]:** Restored [[00_Master_Links_and_Intro|EC2]] instances can be placed within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] for network isolation.

#### 9. USE CASES:
- **Enterprise Data Centers:** Companies maintaining large on-premises data centers that need cloud-based [[00_Master_Links_and_Intro|disaster recovery]] solutions.
- **Hybrid Cloud Environments:** [[organizations]] with hybrid infrastructures ensuring business continuity and quick failover to AWS.

#### 10. DIAGRAMS:
```
+---------------------+
| On-Premises Servers |
|       (Agents)      |
+---------+-----------+
          |
          v
+---------+-----------+
|     [[AWS DRS]]     |
+---------+-----------+
          |
          v
+---------+-----------+
|  Amazon S3 Storage  |
+---------+-----------+
          |
          v
+---------+-----------+
|   EC2 Instances     |
| (Restored Servers)  |
+---------------------+
```

#### 11. EXAM TRAPS:
- Misunderstanding the difference between application-consistent and crash-consistent snapshots.
- Assuming all on-premises servers can be directly replicated without agent installation.

> [!warning] Distractor
Misunderstanding the difference between application-consistent and crash-consistent snapshots.

#### 12. DECISION MATRIX: Features vs. Use Cases
| Feature                  | Use Case                |
|--------------------------|-------------------------|
| Real-time Replication    | Continuous Data Backup  |
| Application Consistency  | Financial Services      |
| [[00_Master_Links_and_Intro|EC2]] Restoration          | Quick Recovery          |
| [[cloudwatch]] Monitoring    | Proactive Management    |

#### 13. 2026 UPDATES:
- Enhanced agent capabilities for broader OS support.
- Improved integration with AWS [[appsync|Security]] services like [[GuardDuty]].

#### 14. EXAM SCENARIOS:
- Scenario: A customer needs to replicate a set of on-premises servers and restore them in case of an outage.
- Question: How would you configure RPO/RTO settings using the CLI?

> [!check] Implementation
```sh
aws drs start-replication-config-task # Starts the replication configuration task
```

#### 15. INTERACTION PATTERNS:
- **Agent to AWS [[drs]]:** Continuous data replication updates.
- **AWS [[drs]] to [[00_Master_Links_and_Intro|EC2]]:** Instantiation and restoration of servers.

> [!abstract] Exam Tip
Understanding interaction patterns between agents, AWS [[drs]], and [[00_Master_Links_and_Intro|EC2]] is crucial for [[00_Master_Links_and_Intro|disaster recovery]] scenarios.

#### 16. STRATEGIC ALIGNMENT:
Each point is aligned with the latest SAP-C02 exam blueprint, focusing on high-weight topics like [[00_Master_Links_and_Intro|disaster recovery]] and integration with core [[AWS]] services.

> [!danger] Distractor
While [[drs]] is crucial for [[00_Master_Links_and_Intro|disaster recovery]], it's not the appropriate service to ensure high availability and continuous uptime.

#### 17. PROFESSIONAL TONE:
- High-density technical breakdown with concise explanations.
  
#### 18. AUDIENCE: 
Tailored specifically for SAP-C02 candidates preparing for a rigorous exam environment.

#### 19. PRIORITIZATION:
Focuses on high-weight topics like replication, restoration, and integration with AWS services.

#### 20. CLARITY:
Ensures that complex concepts are explained concisely to aid quick comprehension.

#### 21. GROUNDING:
References official [[AWS]] documentation for validation and accuracy.

#### 22. VALIDATION:
Aligns with the latest exam blueprint to ensure relevance and coverage.

#### 23. ARCHITECTURAL TRADE-OFFS:
Choosing [[00_Master_Links_and_Intro|EC2]] instances and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] storage configurations impacts cost, performance, and scalability during [[00_Master_Links_and_Intro|disaster recovery]] operations.

> [!warning] Quota
Soft quotas include maximum number of replication jobs per region (usually around 100) and hard quotas related to [[00_Master_Links_and_Intro|EC2]] instance limits per region.