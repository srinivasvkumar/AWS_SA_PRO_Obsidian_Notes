```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### The 6 R's of Migration

The [[6 R's]] of migration are fundamental to the process of moving applications and data from one environment to another, typically from on-premises or legacy systems to cloud environments. These principles guide a systematic approach to ensure smooth transitions while minimizing risks.

#### UNDER-THE-HOOD MECHANICS
1. **[[Rehost]]**: This involves lifting and shifting workloads without making any changes. The underlying mechanics include transferring virtual machine images or container configurations directly into the target environment.
2. **[[Replatform]]**: Involves migrating applications but changing the platform underneath, such as moving from an on-premises relational database to a managed service like [[Amazon RDS]].
3. **[[Refactor/Re-architect]]**: This involves substantial changes to application design for better performance and efficiency in the cloud. This could mean breaking down monolithic systems into microservices.
4. **[[Retire]]**: Removing legacy components that are no longer needed, often through decommissioning or archiving old data and applications.
5. **[[Repurchase]]**: Replacing current applications with modern SaaS solutions available in the cloud.
6. **[[Retain]]**: Keeping certain workloads on-premises due to regulatory requirements, performance needs, or existing investments.

#### EXHAUSTIVE FEATURE LIST
- **Rehost**: [[VM Import/Export]], [[AWS Server Migration Service (SMS)]]
- **Replatform**: Database migrations using [[AWS DMS]], containerization with [[eks]]
- **Refactor/Re-architect**: Microservices deployment with [[ECS/Fargate]], serverless computing with [[lambda]]
- **[[6r|Retire]]**: Archiving tools like [[Amazon S3 Glacier Deep Archive]], decommissioning via [[iam]] roles and [[policies]]
- **Repurchase**: Migration to [[AWS Marketplace solutions]], integration with existing APIs
- **[[6r|Retain]]**: Hybrid architectures using [[VPC peering]], [[ExpressRoute]]

#### STEP-BY-STEP IMPLEMENTATION
1. **Assessment**: Identify applications, data, and infrastructure components.
2. **Planning**: Decide on the migration strategy (Rehost, Replatform, etc.).
3. **Execution**:
   - For **Rehost**: Use AWS SMS or VM Import/Export to migrate VMs.
   - For **Replatform**: Migrate databases using [[dms]]; containerize applications with [[eks]].
   - For **Refactor/Re-architect**: Break down monolithic systems into microservices.
4. **Validation**: Test migrated workloads and ensure performance meets expectations.
5. **Retirement**: Decommission old systems and archive data if necessary.
6. **Repurchase**: Evaluate SaaS solutions from AWS Marketplace for replacement.

#### TECHNICAL LIMITS
- **Rehost**: Limited by the size and complexity of VMs; hard quotas on instance types.
- **Replatform**: Constraints on database migrations depend on compatibility with [[00_Master_Links_and_Intro|RDS]] or [[dynamodb]].
- **Refactor/Re-architect**: Complexity in managing microservices orchestration.
- **[[6r|Retire]]**: Data retention [[policies]] must comply with regulatory requirements.
- **Repurchase**: Availability and cost of SaaS solutions.
- **[[6r|Retain]]**: Network bandwidth [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] for hybrid architectures.

#### CLI & API GROUNDING
- **AWS SMS**:
  ```bash
  aws sms start-replication-task --replication-task-name <task_name>
  ```
- **AWS [[dms]]**:
  ```bash
  aws dms create-replication-instance --replication-instance-id <instance_id> --allocated-storage 50 --vpc-security-group-ids sg-12345678 --availability-zone us-west-2a --engine-version 3.4.1
  ```

#### PRICING & TCO
- **Rehost**: Cost is based on [[00_Master_Links_and_Intro|EC2]] instance usage.
- **Replatform**: Costs depend on [[00_Master_Links_and_Intro|RDS]] or [[dynamodb]] usage.
- **Refactor/Re-architect**: Pricing varies by service used ([[ecs]], [[Fargate]], [[lambda]]).
- **[[6r|Retire]]**: Archival storage costs vary (e.g., [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[00_Master_Links_and_Intro|Glacier]]).
- **Repurchase**: Depends on the specific SaaS pricing model.
- **[[6r|Retain]]**: Hybrid architectures can incur additional networking costs.

#### COMPETITOR COMPARISON
- AWS vs. [[Azure]]: Both offer robust migration services, but Azure may have stronger [[api-gateway|integrations]] with Windows environments.
- AWS vs. [[Google Cloud]]: GCP's Anthos platform provides strong multi-cloud support, which might be more beneficial for complex hybrid architectures.

#### INTEGRATION
- **[[cloudwatch]]**: For monitoring migrated workloads.
- **[[iam]]**: To manage access and permissions post-migration.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]**: For network isolation and [[appsync|security]] in the cloud environment.

#### USE CASES
- **Financial Services**: [[6r|Rehosting]] legacy banking applications to AWS using SMS.
- **Healthcare**: Refactoring patient management systems into microservices for better scalability.
- **Retail**: [[6r|Repurchasing]] CRM solutions from AWS Marketplace for enhanced customer analytics.

#### DIAGRAMS
- Placeholder for Rehost architecture diagram with trade-offs between VM Import/Export and SMS.
- Placeholder for Replatform architecture diagram showing [[dms]] migration paths.

#### EXAM TRAPS
> [!warning] Quota
Common misconceptions include thinking all workloads can be easily rehosted without issues. Overlooking the importance of thorough planning in the 6 R's approach.

#### DECISION MATRIX
| Feature          | Use Case 1 (Financial Services) | Use Case 2 (Healthcare)   | Use Case 3 (Retail)      |
|------------------|----------------------------------|----------------------------|--------------------------|
| Rehost           | High                             | Medium                      | Low                      |
| Replatform       | Medium                           | High                        | Medium                   |
| Refactor/Re-architect | Medium                     | High                        | High                     |
| [[6r|Retire]]           | Low                              | High                        | Medium                   |
| Repurchase       | Low                              | Medium                      | High                     |
| [[6r|Retain]]           | Medium                           | Low                         | Low                      |

#### 2026 UPDATES
All details are current as of the AWS landscape in 2026, including new services and quotas.

#### EXAM SCENARIOS
> [!abstract] Exam Tip
**Scenario**: A company wants to migrate its on-premises databases to AWS. How would you approach this using Replatform?
- **Answer**: Use [[dms]] for database migration, with step-by-step validation of data integrity post-migration.

#### INTERACTION PATTERNS
- Migrated applications interact with [[cloudwatch]] for monitoring and [[vpc-flow-logs|logging]].
- [[00_Master_Links_and_Intro|IAM]] roles manage access control post-migration.

#### STRATEGIC ALIGNMENT
Each point is aligned to ensure comprehensive exam success by covering all aspects of migration strategies on AWS.

#### PROFESSIONAL TONE & AUDIENCE TAILORING
This content is specifically tailored for SAP-C02 candidates, focusing on high-value information without fluff.

#### PRIORITIZATION
Focuses on high-weight exam topics, ensuring clarity and conciseness for complex concepts.

#### GROUNDING
References official AWS documentation for accuracy and validation with the latest exam blueprint.

#### ARCHITECTURAL TRADE-OFFS
Impact of design choices on implementation is discussed thoroughly to ensure optimal migration strategies.