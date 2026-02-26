```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Disaster Recovery (DR) Strategy: Warm Standby

#### Under-the-Hood Mechanics:
Warm standby involves maintaining a secondary environment that is partially active and capable of taking over with minimal manual intervention. The primary system continuously replicates data to the [[warm standby]] site, which runs in a reduced capacity mode until it needs to take over.

- **Data Replication:** Data replication can be synchronous or asynchronous, but for warm standby, asynchronous is more common due to its lower latency impact.
- **Infrastructure Synchronization:** Regular synchronization of infrastructure configurations (like [[security groups]], [[IAM roles]]) ensures that the secondary environment can seamlessly take over in case of a disaster.
- **Consistency Models:** The consistency model is typically eventual consistency since data replication is usually asynchronous. This means there could be some delay before all data is fully propagated to the warm standby.

#### Exhaustive Feature List:
1. **Data Replication:** Continuous or near-continuous data transfer from primary to secondary.
2. **Partial Activation:** The secondary site runs a subset of services and can be scaled up quickly.
3. **Manual Switching:** Manual intervention is required to switch over, but the process should be streamlined for quick recovery.
4. **Infrastructure Synchronization:** Automated tools that ensure configurations are in sync between primary and secondary sites.
5. **Failover Testing:** Regular testing and validation of failover processes.

#### Step-by-Step Implementation:
1. **Assessment & Planning:**
   - Identify critical applications and data.
   - Define recovery time objectives (RTO) and recovery point objectives (RPO).

2. **Setup Secondary Environment:**
   - Deploy [[AWS]] infrastructure in a different region or Availability Zone.
   - Configure IAM roles, security groups, and other necessary resources.

3. **Data Replication Setup:**
   - Use services like [[AWS DMS]] for database replication.
   - Enable S3 cross-region replication for data storage.

4. **Infrastructure Synchronization:**
   - Automate the synchronization of configurations using tools like [[CloudFormation]] or [[Terraform]].

5. **Testing & Validation:**
   - Conduct regular failover tests to ensure processes work as expected.
   - Validate data integrity and application functionality in the secondary environment.

#### Technical Limits (2026):
- **Soft Quotas:** Limited number of instances or resources that can be launched simultaneously during a failover.
- **Hard Quotas:** Enforced limits on S3 bucket replication destinations, number of DMS replication tasks, etc. Refer to [AWS Service Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html).

#### CLI & API Grounding:
- **DMS Replication Setup:**
  ```sh
  aws [[dms]] create-replication-task --replication-task-identifier my-dms-task \
    --source-endpoint-arn arn:aws:[[dms]]:region:account-id:endpoint/source-endpoint \
    --target-endpoint-arn arn:aws:[[dms]]:region:account-id:endpoint/target-endpoint \
    --replication-instance-arn arn:aws:[[dms]]:region:account-id:instance/replication-instance
  ```
- **S3 Cross-region Replication:**
  ```sh
  aws s3api put-bucket-replication --bucket my-source-bucket \
    --replication-configuration file://./cross-region-replication-config.json
  ```

#### Pricing & TCO:
- **Cost Drivers:** Data transfer costs, instance usage in the secondary region, and storage costs.
- **Optimization Strategies:**
  - Use reserved instances or spot instances for cost savings.
  - Implement data lifecycle policies to reduce storage costs.

#### Competitor Comparison:
- **AWS vs. Azure:** Azure offers [[Site Recovery]] for disaster recovery but may require more manual configuration compared to AWS DMS and CloudFormation.
- **Architectural Trade-offs:**
  - Warm standby in AWS allows for flexible scaling, whereas some competitors might have stricter limits.

#### Integration:
- **CloudWatch:** Monitor replication tasks and set up alarms.
- **IAM:** Use roles to manage access to secondary resources.
- **VPC:** Ensure VPC peering or inter-region connectivity is properly configured.

#### Use Cases:
1. **Financial Services:**
   - High availability for transactional systems with minimal data loss during failover.
2. **Healthcare:**
   - Ensuring critical patient information and applications are accessible even in a disaster scenario.

#### Decision Matrix
| Feature             | Use Case               |
|---------------------|------------------------|
| Data Replication    | Financial Transactions  |
| Partial Activation  | Healthcare Applications |
| Manual Switching    | Business Continuity     |

#### Exam Traps:
> [!danger] Distractor
Misunderstanding of the difference between synchronous and asynchronous replication impacts consistency guarantees.
- Confusion about the roles of different AWS services in disaster recovery (e.g., [[dms]] vs. S3 Replication).

### Audit for Disaster Recovery (DR) Strategy: Warm Standby

#### Overview:
A warm standby disaster recovery strategy involves maintaining a partially operational secondary environment that can be quickly brought to full functionality in the event of an outage at the primary site.

### Distractor Analysis:
1. **Fully Active Hot Standby**:
   > [!danger] Distractor
   A warm standby is not suitable when real-time synchronization and zero data loss are required because the secondary environment in a warm standby does not continuously replicate all changes.
   
2. **Cold Standby**:
   > [!warning] Quota
   This option involves minimal operational overhead but requires significant time to bring services online after failover.

3. **Hybrid Cloud Deployments with High Data Synchronization Requirements**:
   > [!danger] Distractor
   Warm standby may not be optimal as it does not provide continuous data synchronization, which might be necessary in hybrid cloud setups where real-time access to the latest data across both environments is critical.
   
4. **Scalability Constraints**:
   > [!danger] Distractor
   If your application requires near-instant scalability and high availability without any downtime, warm standby may not suffice as the partial operational nature of its secondary environment might introduce delays or gaps.

5. **Cost-Sensitive Environments with Minimal Recovery Point Objectives (RPO)**:
   > [!danger] Distractor
   Warm standby does not offer a zero RPO, which means it is less suitable for use cases where minimal data loss is critical.

### The "Most Correct" Logic:
- **Performance vs. Cost**: A warm standby offers faster failover times and partial operational capabilities compared to cold standbys but requires more resources than cold standbys.
- **Operational Complexity**: While it is less complex than fully active hot standby solutions, warm standby still involves managing two environments and ensuring data consistency between them.

### Enterprise Governance:
1. **AWS Organizations**:
   > [!check] Implementation
   Utilize AWS Organizations to centrally manage multiple accounts and apply consistent policies across the entire organization for both primary and secondary environments.
2. **Service Control Policies (SCPs)**:
   > [!check] Implementation
   Implement SCPs to enforce governance controls, restricting actions that could inadvertently disrupt the DR setup or violate organizational compliance requirements.
3. **Resource Access Manager (RAM)**:
   > [!check] Implementation
   Use RAM to share resources such as VPC subnets, NAT gateways, and IAM roles between accounts, simplifying resource management across the primary and secondary environments.
4. **CloudTrail**:
   > [!check] Implementation
   Enable AWS CloudTrail to log API calls and operational activities for both environments, facilitating audit trails and compliance monitoring.

### Tier-3 Troubleshooting:
1. **KMS Key Policy Conflicts**: Ensure that KMS key policies are correctly configured in both environments to avoid access issues when failover occurs.
2. **Network Configuration Issues**: Verify VPC peering connections, route tables, and security groups between primary and secondary environments to prevent network-related failures during failover.
3. **Data Consistency Problems**: Address potential data inconsistency by implementing appropriate replication mechanisms or using AWS services like [[AWS Database Migration Service (DMS)]] for consistent state synchronization.

### Decision Matrix: Use Case vs. Solution
| Use Case                          | Warm Standby Suitability | Rationale                                                                 |
|-----------------------------------|--------------------------|---------------------------------------------------------------------------|
| High Availability Required        | Moderate                 | Partially operational secondary environment; faster recovery than cold standbys but not as fast as hot standbys. |
| Cost-Sensitive                    | High                     | Less resource-intensive compared to fully active environments, making it suitable for cost-sensitive scenarios.    |
| Near-Real Time Data Consistency   | Moderate                 | Requires manual or automated processes to ensure data consistency between environments.                       |
| Regulatory Compliance             | High                     | Central management via AWS Organizations and SCPs helps in maintaining compliance across both environments.      |

### Recent Updates:
1. **General Availability (GA) Features**: 
   - New features for [[AWS DMS]] and other services that enhance replication and failover capabilities.
2. **Deprecated Items for 2026**:
   - Older versions of certain AWS services or specific API endpoints might be deprecated in favor of more modern and efficient alternatives.

```