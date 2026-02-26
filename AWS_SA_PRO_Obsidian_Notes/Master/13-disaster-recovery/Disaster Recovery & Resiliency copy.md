```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Disaster Recovery]] and Resiliency

#### Under-the-Hood Mechanics:
[[Disaster Recovery (DR)]] in [[AWS]] involves multiple services such as [[Amazon RDS]], [[Amazon EBS]], [[AWS Backup]], [[AWS Lambda]], [[cloudformation]], [[Route 53]], and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]. The underlying infrastructure logic includes:

- **Data Replication:** Data is replicated across Availability Zones ([[AZs]]) or Regions using synchronous/asynchronous mechanisms.
- **Consistency Models:** Point-in-time recovery for [[rds]], [[00_Master_Links_and_Intro|EBS]] snapshots for consistent state of data at a given time.
- **Failover Mechanisms:** [[Route 53]] [[route53|health checks]] trigger failovers to standby instances in another AZ or Region.

#### Exhaustive Feature List:
1. **Cross-Region Replication** (CRS) - For [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[00_Master_Links_and_Intro|RDS]].
2. **Amazon Elastic Block Store ([[ebs]])** Snapshots for stateful applications.
3. **[[00_Master_Links_and_Intro|AWS Backup]]** - Centralized backup management across multiple services.
4. **[[lambda|AWS Lambda]]** - Automated failover logic using serverless functions.
5. **[[cloudformation]] [[cloudformation|StackSets]]** - Automate [[dr]] stack deployment in different Regions.
6. **[[00_Master_Links_and_Intro|Route 53]] DNS Failover** - Routing traffic to healthy endpoints.
7. **[[Amazon Aurora Global Database]]** - Automatic replication and read replicas.

#### Step-by-Step Implementation:
1. **Assessment:** Identify critical applications, data sources, and recovery time objectives (RTOs).
2. **Backup Strategy:** Use [[00_Master_Links_and_Intro|AWS Backup]] for consistent backups across multiple services.
3. **Replication Setup:** Enable CRS for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[00_Master_Links_and_Intro|EBS]] snapshots for [[00_Master_Links_and_Intro|RDS]] instances.
4. **Automated Failover:** Configure [[00_Master_Links_and_Intro|Route 53]] [[route53|health checks]] and failovers with [[cloudformation]] templates.
5. **Testing & Maintenance:** Regularly test [[dr]] plans using [[cloudformation]] [[cloudformation|StackSets]].

#### Technical Limits:
- **[[00_Master_Links_and_Intro|EBS]] Snapshot Limits:** 1,000 snapshots per region, 25 concurrent snapshot creations.
- **[[00_Master_Links_and_Intro|RDS]] Replication Limits:** Maximum of five read replicas per instance, up to three DB instances in a multi-AZ deployment.
- **[[lambda]] Execution Limits:** 1 million requests per account per month.

#### CLI & API Grounding:
```sh
aws rds create-db-instance-read-replica
aws backup start-backup-job
aws route53 change-resource-record-sets
```

#### Pricing & TCO:
- **Storage Costs:** Increased storage costs due to replicated data in multiple Regions.
- **Compute Costs:** Additional compute resources for standby instances.
- **Optimization Strategies:** Use [[00_Master_Links_and_Intro|reserved instances]], [[00_Master_Links_and_Intro|spot instances]], and tiered backup [[policies]].

#### Competitor Comparison:
- **Azure:** Azure Site Recovery vs. [[00_Master_Links_and_Intro|AWS Backup]] & [[lambda]] for automation.
- **Google Cloud Platform ([[GCP]])**: GCP’s Compute Engine snapshots and Google Cloud Storage vs. AWS [[00_Master_Links_and_Intro|EBS]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] CRS.

#### Integration:
- **[[cloudwatch]]:** Monitoring [[route53|health checks]] and failover processes.
- **[[00_Master_Links_and_Intro|IAM]] Roles:** Assigning roles with least privilege access to backup, replication tasks.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering & [[Transit gateway]]:** Securing network connectivity between primary and [[dr]] Regions.

#### Use Cases:
1. **E-commerce Platform:** Automated failover using [[00_Master_Links_and_Intro|Route 53]] for minimal downtime during disaster scenarios.
2. **Healthcare Application:** Multi-AZ [[00_Master_Links_and_Intro|RDS]] instances with regular [[00_Master_Links_and_Intro|EBS]] snapshots to ensure data availability and compliance.

#### Decision Matrix:
| Features                 | Use Case: E-commerce Platform   | Use Case: Healthcare Application |
|--------------------------|---------------------------------|-----------------------------------|
| Multi-AZ Deployments     | High Availability               | Compliance & Data Integrity       |
| Automated Failovers      | Minimized Downtime              | Rapid Recovery                    |
| Regular Backups          | Business Continuity Planning    | Regulatory Compliance             |

#### Exam Traps:
1. Misunderstanding the difference between synchronous vs asynchronous replication.
2. Overlooking the importance of regular failover testing.

#### 2026 Updates:
- Enhanced [[00_Master_Links_and_Intro|AWS Backup]] features for cross-account backup management.
- Improved [[lambda]] concurrency controls for failover automation.

#### Exam Scenarios:
1. **Question:** How would you ensure minimal downtime during a region-wide disaster?
   - **Answer:** Use [[Route 53]] DNS Failover with [[cloudformation]] [[cloudformation|StackSets]] to automate [[dr]] process.
   
2. **Question:** What service would you use to regularly back up data across multiple services?
   - **Answer:** [[00_Master_Links_and_Intro|AWS Backup]] for centralized backup management.

#### Interaction Patterns:
- [[00_Master_Links_and_Intro|Lambda functions]] triggering failover logic.
- Synchronous [[00_Master_Links_and_Intro|EBS]] snapshots before initiating failover.

---

### 1. Distractor Analysis

To identify scenarios where a specific AWS service may be the wrong answer for [[00_Master_Links_and_Intro|disaster recovery]] and resiliency, consider:

> [!danger] Distractor
- **Scenario 1: Cold Storage Backup**  
  If the primary requirement is to store long-term backups that are rarely accessed, using services like [[Amazon S3]] with [[00_Master_Links_and_Intro|Glacier]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] would be more appropriate than AWS [[00_Master_Links_and_Intro|RDS]] automated backups or [[00_Master_Links_and_Intro|EBS]] Snapshots.

> [!danger] Distractor
- **Scenario 2: Real-Time Data Replication Across Regions**  
  For real-time data replication across regions, AWS Database Migration Service ([[dms]]) is not the best fit. Instead, services like [[00_Master_Links_and_Intro|Amazon Aurora]] Global Databases are designed to provide cross-region replication with minimal latency and automatic failover capabilities.

> [!danger] Distractor
- **Scenario 3: Low-Bandwidth Environments**  
  In scenarios where bandwidth is limited, using high-bandwidth consuming services such as [[AWS CloudFormation]] or [[cicd|AWS CodePipeline]] for [[00_Master_Links_and_Intro|disaster recovery]] might not be optimal. Instead, leveraging services like [[00_Master_Links_and_Intro|AWS Backup]] (with selective data backup) or even manual script-based backups can reduce the load on network resources.

> [!danger] Distractor
- **Scenario 4: On-Premises to AWS Migration**  
  If the goal is an efficient and automated migration from on-premises infrastructure to [[AWS]], tools like [[AWS Server Migration Service (SMS)]] are more appropriate compared to manually exporting data and importing it into services like Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[00_Master_Links_and_Intro|EBS]].

> [!danger] Distractor
- **Scenario 5: Legacy Application Support**  
  For legacy applications that do not support modern [[00_Master_Links_and_Intro|disaster recovery]] techniques, using cutting-edge services such as [[AWS Lambda]] or [[ecs]] may introduce significant integration overhead. Instead, traditional RTO/RPO solutions using [[00_Master_Links_and_Intro|EC2]] with Auto Scaling and [[cloudwatch]] would be more straightforward.

### 2. The "Most Correct" Logic

When selecting a service for [[00_Master_Links_and_Intro|disaster recovery]] and resiliency:

> [!warning] Quota
- **Performance vs. Cost**: 
  - High-performance options such as [[00_Master_Links_and_Intro|Amazon Aurora]] with Global Databases provide low RTO/RPO but may be costly.
  - Budget-friendly solutions like [[00_Master_Links_and_Intro|AWS Backup]] and manual replication scripts have higher RTO/RPO but are more economical.

### 3. Enterprise Governance

To ensure compliance and [[appsync|security]] in [[00_Master_Links_and_Intro|disaster recovery]]:

> [!abstract] Exam Tip
- **[[organizations|AWS Organizations]]**: Use [[organizations]] to manage multiple accounts and services under a single entity, facilitating centralized management and control.
  
- **Service Control [[policies]] (SCPs)**: Implement SCPs to enforce [[policies]] across all member accounts, ensuring that only authorized actions are performed on critical resources like [[00_Master_Links_and_Intro|disaster recovery]] plans.

- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Use [[ram]] to share [[00_Master_Links_and_Intro|disaster recovery]] resources such as [[00_Master_Links_and_Intro|AWS Backup]] vaults or [[cloudformation]] templates across multiple [[AWS]] accounts within your organization.

- **[[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to log API calls and user activity, providing an audit trail that can help in identifying any unauthorized actions or suspicious activities during the [[00_Master_Links_and_Intro|disaster recovery]] process.

### 4. Tier-3 Troubleshooting

Document complex failure modes for [[00_Master_Links_and_Intro|disaster recovery]]:

> [!check] Implementation
- **[[kms]] Key Policy Conflicts**: 
  - If [[kms]] keys are used to encrypt backups and [[00_Master_Links_and_Intro|disaster recovery]] data, conflicts in key [[policies]] can prevent decryption upon failover.
  - Ensure that [[kms]] key [[policies]] are consistent across all regions and accounts involved in the [[00_Master_Links_and_Intro|disaster recovery]] process. Use SCPs to enforce proper policy configurations.

### 5. Decision Matrix

Create a "Use Case vs. Solution" table for [[00_Master_Links_and_Intro|disaster recovery]]:

| **Use Case**                  | **Solution**                      | **RTO (Recovery Time Objective)** | **RPO (Recovery Point Objective)** |
|-------------------------------|----------------------------------|-----------------------------------|-----------------------------------|
| On-premises to AWS Migration  | AWS Server Migration Service     | Minutes                            | Seconds                            |
| Cross-Region Replication      | [[00_Master_Links_and_Intro|Amazon Aurora]] Global Databases   | Seconds                             | Zero                               |
| Long-term Backup Storage      | Amazon [[Master/Srinivas_Notes/S3|S3]] with [[00_Master_Links_and_Intro|Glacier]]           | Hours                              | Days                               |
| Automated Backups             | [[00_Master_Links_and_Intro|AWS Backup]]                       | Minutes to Hours                   | Minutes to Hours                    |

### 6. Recent Updates

Flag any GA features or deprecated items for 2026:

- **GA Features**: 
  - As of the latest updates, new [[00_Master_Links_and_Intro|disaster recovery]] features like enhanced cross-region replication capabilities in [[Amazon RDS]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering for [[00_Master_Links_and_Intro|disaster recovery]] data storage have been introduced.
  
- **Deprecated Items**:
  - AWS recommends transitioning from older services such as [[dynamodb]] Streams for real-time data replication to newer alternatives like Amazon [[eventbridge]].

By following these enhancements, you ensure that the audit provides a comprehensive and accurate analysis of [[00_Master_Links_and_Intro|disaster recovery]] and resiliency in an [[AWS]] environment.