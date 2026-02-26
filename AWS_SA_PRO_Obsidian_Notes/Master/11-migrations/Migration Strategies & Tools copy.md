```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[AWS]] Migration Strategies & Tools

#### UNDER-THE-HOOD MECHANICS:
Migration strategies and tools on AWS encompass a range of services that facilitate the movement of workloads from on-premises or other cloud environments to AWS. The core mechanics involve data replication, application migration, and infrastructure orchestration.

1. **Data Replication**: Services like [[AWS Database Migration Service (DMS)]] use change data capture (CDC) techniques to synchronize databases between source and target.
2. **Application Migration**: Tools such as [[VM Import/Export]] allow the conversion of virtual machines from various formats into Amazon Machine Images (AMIs).
3. **Infrastructure Orchestration**: [[AWS Migration Hub]] provides a dashboard for managing migrations, orchestrating tasks across multiple services.

#### EXHAUSTIVE FEATURE LIST:
1. **[[AWS Database Migration Service (DMS)]]**:
   - Real-time replication.
   - Schema conversion.
   - CDC.
2. **[[VM Import/Export]]**:
   - Convert and import VMs to AMIs.
   - Export EC2 instances as VMs.
3. **[[Application Discovery Service]]**:
   - Inventory on-premises resources.
   - Analyze workloads for cloud readiness.
4. **[[Migration Hub]]**:
   - Centralized migration dashboard.
   - Task orchestration.
5. **[[Server Migration Service (SMS)]]**:
   - Migrate physical and virtual servers to EC2.
6. **[[Storage Gateway]]**:
   - Connect on-premises storage with AWS.
7. **[[AWS Schema Conversion Tool (SCT)]]**:
   - Convert database schemas between different types.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Discovery**: Use [[Application Discovery Service]] to assess your environment and identify workloads for migration.
2. **Replication**: Set up [[DMS]] or [[SMS]] to replicate data from on-premises environments.
3. **Conversion**: Utilize VM Import/Export for converting virtual machines into AMIs.
4. **Migration**: Deploy resources using Migration Hub, which orchestrates tasks across multiple services.
5. **Validation**: Ensure the migrated applications and databases are functioning as expected in AWS.

#### TECHNICAL LIMITS (2026):
1. **[[DMS]]**:
   - Maximum of 100 replication instances per region.
   - Up to 3,840 tasks per replication instance.
2. **[[Migration Hub]]**:
   - Limited number of concurrent tasks.
3. **[[VM Import/Export]]**:
   - Disk size limits (e.g., up to 16 TB for single disks).

#### CLI & API GROUNDING:
- **DMS**: `aws dms create-replication-instance`, `aws dms create-endpoint`
- **Migration Hub**: `aws MigrationHub start-discovery`
- **VM Import/Export**: `aws ec2 import-image`

#### PRICING & TCO:
1. **[[DMS]]**: Pay per replication instance and data transfer.
2. **SMS**: Free service, pay for EC2 usage during migration.
3. **Storage Gateway**: Volume-based pricing.

**Optimization Strategies**:
- Use reserved instances or savings plans to reduce costs.
- Optimize storage by using lifecycle policies in S3.

#### COMPETITOR COMPARISON:
1. **[[Azure Migrate]] vs [[DMS]], [[SMS]]**:
   - Azure Migrate offers similar discovery and assessment features but lacks the broad range of replication tools compared to AWS DMS.
2. **Google Cloud Migrator vs Migration Hub**:
   - Google’s service integrates well with their ecosystem but lacks some orchestration features found in AWS Migration Hub.

#### INTEGRATION:
1. **[[CloudWatch]]**: Monitor migration tasks and resource usage.
2. **IAM**: Manage access to migration services through roles and policies.
3. **VPC**: Deploy migrated resources within VPCs for network isolation.

#### USE CASES:
1. **Enterprise Data Warehouse Migration**:
   - Use DMS to migrate a large data warehouse from Oracle to Amazon Redshift.
2. **Application Modernization**:
   - Migrate virtualized applications to AWS using VM Import/Export and then containerize them with ECS or EKS.

#### DIAGRAMS:
- Placeholder for architectural diagram showing migration flow from on-premises to AWS, including Application Discovery Service, DMS, and Migration Hub interactions.

#### EXAM TRAPS:
1. **Misunderstanding Replication vs Conversion**:
   - Candidates often confuse the roles of services like DMS (replication) and VM Import/Export (conversion).
2. **Overlooking Pricing Details**:
   - Be aware of usage-based pricing for migration tools, particularly with DMS.

#### DECISION MATRIX: Features vs. Use Cases
| Feature                | Use Case 1: Data Warehouse Migration | Use Case 2: Application Modernization |
|------------------------|--------------------------------------|---------------------------------------|
| **[[DMS]]**            | High                                | Medium                                 |
| **VM Import/Export**   | Low                                 | High                                   |
| **Migration Hub**      | Medium                              | High                                   |

#### EXAM SCENARIOS:
- Scenario: Migrating a large database from Oracle to Amazon Aurora.
- Question: Which tool would you use for real-time data replication?

#### INTERACTION PATTERNS:
1. **DMS -> RDS**: Continuous replication of on-premises databases to AWS-managed services.
2. **VM Import/Export -> EC2**: Conversion and deployment of virtual machines as EC2 instances.

### Audit for Migration Strategies & Tools

#### Distractor Analysis
Here are some scenarios where specific migration strategies or tools may not be the right choice:

- **Scenario 1: Using [[AWS Server Migration Service (SMS)]] for migrating small databases**
  - SMS is designed to migrate large on-premises servers and workloads, making it inefficient and potentially overkill for smaller database migrations.

- **Scenario 2: Choosing AWS Application Discovery Service for a simple migration of a single application**
  - The [[Application Discovery Service]] is more suitable for complex environments where comprehensive analysis of applications and dependencies is required. For simpler scenarios, manual analysis might be sufficient.

- **Scenario 3: Utilizing AWS Database Migration Service (DMS) to migrate non-relational databases to Amazon RDS**
  - DMS supports relational database migrations but not all non-relational data sources effectively. In such cases, other tools like [[AWS Data Pipeline]] or custom scripts may be more appropriate.

- **Scenario 4: Employing AWS CloudFormation for ad-hoc infrastructure deployment during migration**
  - [[CloudFormation]] is best suited for repeatable, automated deployments and managing infrastructure as code (IaC). Ad-hoc changes should typically use the console or API directly to avoid unnecessary complexity.

#### The "Most Correct" Logic: Trade-offs
- **Performance vs. Cost**: 
  - **High Performance**: Tools like [[AWS SMS]] and DMS are optimized for high-speed migrations, often at a higher cost due to their specialized nature.
  - **Cost-Efficient**: For smaller workloads or less time-sensitive migrations, manual methods or lightweight tools such as the AWS Management Console might be more economical.

#### Enterprise Governance
- **[[AWS Organizations]]**:
  - Use AWS Organizations for managing multiple accounts and services centrally.
  
- **Service Control Policies (SCPs)**:
  - Implement SCPs to enforce organizational policies at different levels within the organization hierarchy, restricting access or actions based on predefined rules.

- **Resource Access Manager (RAM)**:
  - Leverage RAM to share resources like VPCs, IAM roles, and security groups across multiple AWS accounts efficiently.

- **[[AWS CloudTrail]]**:
  - Enable CloudTrail for logging API calls and activity monitoring, ensuring transparency in migration processes and compliance with audit requirements.

#### Tier-3 Troubleshooting
Complex failure modes can arise during migrations:

- **KMS Key Policy Conflicts**: 
  - Ensure that key policies are consistent across different services and accounts to prevent access issues. Misconfigured KMS keys can lead to encrypted data becoming inaccessible post-migration.
  
- **Network Configuration Issues**:
  - Incorrect VPC configurations, subnets, or security groups can disrupt connectivity between migrated resources.

#### Decision Matrix: Use Case vs. Solution
| Use Case                           | Recommended Tool/Service       |
|------------------------------------|-------------------------------|
| Large-scale server migration        | [[AWS Server Migration Service (SMS)]] |
| Application dependency analysis     | AWS Application Discovery Service   |
| Database migrations                 | [[DMS]]  |
| Automated infrastructure deployment | [[CloudFormation]]             |

#### Recent Updates
- **GA Features**:
  - In 2026, expect new features in SMS and DMS that will enhance their performance and usability.
  
- **Deprecated Items**:
  - Older migration tools like AWS Import/Export might be deprecated or phased out as newer services take over.

### Conclusion
Properly selecting and using the right set of migration strategies and tools is essential for a successful transition to AWS. This guide provides insights into common pitfalls, trade-offs, governance considerations, troubleshooting tips, decision-making aids, and recent updates to help achieve optimal results in your cloud migration journey.
```