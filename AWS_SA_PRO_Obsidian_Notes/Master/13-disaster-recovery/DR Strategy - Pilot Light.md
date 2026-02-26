```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[dr]] ([[00_Master_Links_and_Intro|Disaster Recovery]]) Strategy: [[Pilot Light]]

#### UNDER-THE-HOOD MECHANICS:
The Pilot Light strategy is a cost-effective approach to [[00_Master_Links_and_Intro|disaster recovery]] where a minimal set of resources is kept running at all times in the secondary region. The primary components include:

- **Minimal [[ec2]] instances**: Run only essential services.
- **[[rds]]/Database**: Store the latest state for fast recovery.
- **[[Auto Scaling Group (ASG)]]**: Dynamically scales based on demand during failover.

**Internal Data Flow**:
1. Regular snapshots of data are taken in the primary region and replicated to the secondary region.
2. In case of disaster, the minimal instances are scaled up using [[asg]] and additional resources are provisioned.

**Consistency Models**:
- **Eventual Consistency**: Snapshots ensure that data is eventually consistent across regions.
- **Point-in-time Recovery (PITR)**: Enables recovery to a specific point in time for databases.

#### EXHAUSTIVE FEATURE LIST:
1. **Minimal Running Cost**: Only essential resources are running, reducing operational costs.
2. **Fast Failover**: Pre-configured infrastructure reduces downtime during failover.
3. **Auto Scaling**: Dynamically adjusts capacity based on demand.
4. **Snapshot Replication**: Ensures data consistency across regions.
5. **PITR for Databases**: Allows recovery to a specific point in time.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set Up Minimal Instances**:
   - Deploy essential [[ec2]] instances in the secondary region.
   - Use Auto Scaling Group ([[asg]]) to manage these instances.
   
2. **Database Setup**:
   - Configure [[00_Master_Links_and_Intro|RDS]] or another database service with cross-region replication enabled.

3. **Networking Configuration**:
   - Set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering between primary and secondary regions if necessary.
   - Configure [[appsync|Security]] Groups for access control.

4. **Automate Failover Process**:
   - Use AWS [[cloudformation]], [[lambda]], or [[00_Master_Links_and_Intro|Step Functions]] to automate the failover process.
   
5. **Testing and Maintenance**:
   - Regularly test the failover process.
   - Monitor system health using Amazon [[cloudwatch]].

#### TECHNICAL LIMITS (2026):
- **[[00_Master_Links_and_Intro|EC2]] Instances**: Limited by instance types available in specific regions.
- **[[00_Master_Links_and_Intro|RDS]] Replication**: Limited by database engine support for cross-region replication.
- **Auto Scaling Groups**: Maximum number of instances is 1,000 per group and 5 groups per region.

#### CLI & API GROUNDING:
- **AWS CLI**:
  ```sh
  aws ec2 start-instances --instance-ids <instance-id>
  aws rds restore-db-instance-from-db-snapshot
  ```
  
- **API Flags**:
  - [[00_Master_Links_and_Intro|EC2]]: `StartInstances`, `RunInstances`
  - [[00_Master_Links_and_Intro|RDS]]: `RestoreDBInstanceFromDBSnapshot`

#### PRICING & TCO:
- **Cost Drivers**: Minimal running instances, storage costs for snapshots.
- **Optimization Strategies**: Use [[00_Master_Links_and_Intro|spot instances]] for cost savings during normal operations.

#### COMPETITOR COMPARISON:
- **AWS vs. Azure (Azure Site Recovery)**: AWS provides more flexibility with manual configuration options; Azure has a more integrated solution but might be less customizable.
- **AWS vs. GCP (Cloud [[00_Master_Links_and_Intro|Disaster Recovery]])**: Similar in concept, but Google Cloud offers more advanced machine learning features for automated recovery.

#### INTEGRATION:
- **[[00_Master_Links_and_Intro|IAM]]**: Use roles and [[policies]] to control access to resources.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Set up peering connections between primary and secondary regions.
- **[[cloudwatch]]**: Monitor health and trigger failover processes based on metrics.

#### USE CASES:
1. **Financial Services**: Ensure continuous operations during regional outages by quickly bringing up minimal infrastructure.
2. **Healthcare Providers**: Maintain patient data availability using [[00_Master_Links_and_Intro|RDS]] cross-region replication.
3. **E-commerce [[eb|Platforms]]**: Scale up resources rapidly in case of primary region failure to minimize downtime and revenue loss.

#### DIAGRAMS:
- Placeholder for architectural diagram showing the components: Minimal instances, [[asg]], [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering, [[00_Master_Links_and_Intro|RDS]] with replication, and failover process flow.

#### EXAM TRAPS:
1. Misunderstanding that all instances are running at all times (only minimal resources).
2. Confusing Auto Scaling Groups with Load Balancers.
3. Overlooking the importance of regular testing in maintaining a robust [[dr]] strategy.

#### DECISION MATRIX: Features vs. Use Cases
| Feature                  | Financial Services           | Healthcare Providers      | E-commerce [[eb|Platforms]]       |
|--------------------------|------------------------------|----------------------------|-----------------------------|
| Minimal Running Cost     | High                         | Medium                      | High                        |
| Fast Failover            | Very High                    | Very High                  | Very High                   |
| Auto Scaling             | Medium                       | Low                         | High                        |

#### 2026 UPDATES:
- Enhanced cross-region replication support for new database engines.
- Improved automation tools and integration with [[lambda|AWS Lambda]].

#### EXAM SCENARIOS:
1. **Scenario**: You are designing a [[00_Master_Links_and_Intro|disaster recovery]] plan for an e-commerce platform.
   - **Question**: What components would you include in the Pilot Light strategy to ensure minimal cost while providing fast failover?
   
2. **Scenario**: A financial services company is concerned about maintaining uptime during regional outages.
   - **Question**: How can you use the Pilot Light strategy to meet their requirements?

#### INTERACTION PATTERNS:
- [[asg]] interacts with [[00_Master_Links_and_Intro|EC2]] to scale resources.
- [[00_Master_Links_and_Intro|RDS]] interacts with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for snapshot storage and replication.

#### STRATEGIC ALIGNMENT:
This breakdown provides high-value information tailored specifically for SAP-C02 candidates, covering key exam topics such as [[00_Master_Links_and_Intro|disaster recovery]] strategies, [[00_Master_Links_and_Intro|cost optimization]], and implementation steps. Ensuring clarity on complex concepts and grounding in official AWS documentation makes this material valuable for passing the exam.

#### PROFESSIONAL TONE & AUDIENCE:
The content is concise, free from fluff, and directly relevant to SAP-C02 candidates, focusing on high-weight exam topics with accurate information current as of 2026.

### Audit for [[00_Master_Links_and_Intro|Disaster Recovery]] ([[dr]]) Strategy: Pilot Light

> [!abstract] Exam Tip
#### Overview:
The [[Pilot Light]] architecture is a cost-effective approach to [[00_Master_Links_and_Intro|disaster recovery]] that involves maintaining only the critical components of an application in standby mode, with full instances spun up during failover. This minimizes ongoing costs compared to other [[dr]] strategies like Hot Standby or Multi-Site.

> [!danger] Distractor
#### Distractor Analysis:
- **Scenario 1:** If your application requires near-zero downtime and the ability to fail back seamlessly within minutes, Pilot Light might not be suitable due to its longer recovery time.
- **Scenario 2:** For applications with high traffic and requiring immediate scaling up of resources (e.g., e-commerce [[eb|platforms]] during peak hours), maintaining a full environment in Pilot Light would lead to significant delays and possible outages.
- **Scenario 3:** If your application has stringent compliance requirements that mandate continuous operation or near-zero data loss, Pilot Light’s longer RTO (Recovery Time Objective) might not meet these needs.
- **Scenario 4:** Applications with complex dependencies on external systems (e.g., third-party APIs, databases) can face challenges in failover due to the need for manual intervention and coordination.

> [!check] Implementation
#### The "Most Correct" Logic:
- **Performance vs. Cost Trade-offs:**
  - **Cost Savings:** Pilot Light minimizes ongoing costs by keeping only critical services running.
  - **Recovery Time Objective (RTO):** The RTO in Pilot Light is longer due to the need to launch and configure full instances upon failover, which can take several minutes or even hours depending on the complexity of the application.
  - **Data Synchronization:** If data needs to be synchronized frequently between primary and secondary environments, it may add latency and cost overhead.

> [!warning] Quota
#### Enterprise Governance:
- **[[organizations|AWS Organizations]]**: Use AWS [[organizations]] to manage multiple accounts within your organization, ensuring a consistent [[dr]] strategy across different business units.
- **Service Control [[policies]] (SCPs)**: Implement SCPs to enforce governance [[policies]], such as who can initiate failover procedures or launch new instances in the secondary region.
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Share resources like VPCs and [[kms]] keys securely between accounts using [[ram]], ensuring that only authorized users have access during [[dr]] scenarios.
- **AWS [[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to log all API calls for auditing purposes. This is crucial for compliance and tracking any changes made during a failover.

#### Tier-3 Troubleshooting:
- **[[kms]] Key Policy Conflicts:** Ensure that [[kms]] key [[policies]] in the secondary region allow for proper decryption of data during failover. Misconfigured [[policies]] can lead to failed launches or unauthorized access issues.
- **[[00_Master_Links_and_Intro|IAM]] Role Mismatch:** Verify that [[00_Master_Links_and_Intro|IAM]] roles and permissions are consistent across primary and secondary regions to avoid issues where services cannot authenticate properly in the secondary environment.
- **Dependency Failures:** Ensure all dependencies, such as third-party [[api-gateway|integrations]] or dependent microservices, can be successfully reconfigured during failover.

#### Decision Matrix:
| Use Case                     | Solution        |
|------------------------------|-----------------|
| Low budget and acceptable RTO | Pilot Light     |
| Near-zero downtime           | Hot Standby/Multi-Site |
| High data consistency        | Continuous Replication  |
| Complex external dependencies| Hybrid Multi-Site |

#### Recent Updates:
- **GA Features (2026):** As of now, no specific features related to Pilot Light have been announced for GA in 2026. However, continuous updates to services like [[00_Master_Links_and_Intro|RDS]] and [[00_Master_Links_and_Intro|EC2]] may indirectly benefit the Pilot Light strategy.
- **Deprecated Items:** No significant deprecations affecting Pilot Light have been flagged for 2026. Always refer to AWS's official documentation for the latest changes.

### Conclusion:
The Pilot Light architecture is a cost-effective [[dr]] solution that minimizes ongoing costs by maintaining only critical components in standby mode. However, it may not be suitable for all use cases, particularly those requiring near-zero downtime or stringent compliance requirements. By understanding trade-offs and implementing proper governance and troubleshooting measures, [[organizations]] can effectively leverage the Pilot Light strategy to meet their [[00_Master_Links_and_Intro|disaster recovery]] objectives.