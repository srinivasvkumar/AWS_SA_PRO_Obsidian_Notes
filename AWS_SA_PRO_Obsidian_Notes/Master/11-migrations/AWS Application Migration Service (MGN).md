```yaml
tags: [AWS/MGN, DataArchitecture]
status: to_review
```

### [[AWS Application Migration Service (MGN)]]

#### 1. UNDER-THE-HOOD MECHANICS

**Internal Data Flow:**
- **Replication Agent:** [[MGN]] uses a replication agent installed on the source server to capture changes. The agent continuously sends data to the staging area in [[Amazon S3]].
- **Staging Area:** Changes are temporarily stored in an [[S3 bucket]] for processing before being sent to the target environment (e.g., [[EC2 instances]]).
- **Target Environment:** Data is then replicated to the target environment using [[AMIs]] or [[EBS volumes]].

**Consistency Models:**
- [[MGN]] supports various consistency models depending on the type of workload, including full and incremental replication. Full replication captures an initial snapshot, while subsequent changes are captured incrementally.
- Real-time replication ensures that minimal data loss occurs during migration, but it may require more bandwidth.

**Underlying Infrastructure Logic:**
- **Networking:** MGN leverages [[VPCs]] to ensure secure communication between source and target environments.
- **[[appsync|Security]]:** Data in transit is encrypted using [[TLS]], and data at rest can be encrypted using [[AWS KMS]] for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets.

#### 2. EXHAUSTIVE FEATURE LIST

**Core Features:**
- **Migration Workflows:** Automated workflows for migration and replication tasks.
- **Agent Management:** Installation, configuration, and monitoring of the replication agents.
- **Replication Tasks:** Full and incremental data replication from source to target.
- **Migration Dashboard:** Visual dashboard for tracking migration progress.

**Advanced Features:**
- **Multi-AZ Replication:** Replicate across multiple Availability Zones for redundancy.
- **Post-Migration Scripts:** Custom scripts can be executed post-migration to configure the environment.
- **On-Demand Migration:** Ability to migrate data in real-time with minimal downtime.
- **Migration [[policies]]:** Define [[policies]] for how and when replication occurs.

**Overlooked Features:**
- **Error Handling and Retry Logic:** Built-in mechanisms to handle [[api-gateway|errors]] during migration and retry failed tasks.
- **API Support:** Full support for programmatic management of migration tasks using the [[AWS SDKs]] or CLI.

#### 3. STEP-BY-STEP IMPLEMENTATION

1. **Plan Migration:** Identify on-premises servers to migrate, create a staging [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket, and set up target [[00_Master_Links_and_Intro|EC2]] instances in an appropriate [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
2. **Install Replication Agent:** Deploy the [[MGN replication agent]] on each source server.
3. **Configure Source Servers:** Register source servers with MGN and configure replication settings.
4. **Replicate Data:**
   - Perform initial full data transfer to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] staging area.
   - Enable real-time incremental replication for ongoing changes.
5. **Launch Target Instances:**
   - Use the [[MGN console]] or APIs to launch target [[00_Master_Links_and_Intro|EC2]] instances from the replicated data.
6. **Monitor Progress:** Utilize MGN’s dashboard to monitor migration status and troubleshoot issues.
7. **Finalize Migration:** Once all data is successfully migrated, switch traffic to the new environment.

#### 4. TECHNICAL LIMITS

- **Soft Quotas (2026):**
  - Maximum number of source servers per replication group: 500
  - Bandwidth limits for real-time replication are dependent on network speed and can be customized.
  
- **Hard Limits (2026):**
  - Number of AWS accounts that can access MGN in an organization is limited to 10.

#### 5. CLI & API GROUNDING

**AWS CLI Commands:**
```sh
# Create a replication job
aws mgn create-replication-job --source-server-id <server_id> --replication-type "REPLICA" --target-instance-type-name "t2.micro"

# List all source servers
aws mgn describe-source-servers

# Start test instance launch
aws mgn start-test-instance-launch --source-server-id <server_id>
```

**API Flags:**
- `CreateReplicationJob`: Specifies the replication type and target instance settings.
- `DescribeSourceServers`: Retrieves detailed information about each source server.

#### 6. PRICING & TCO

**Cost Drivers:**
- **Compute Costs:** [[00_Master_Links_and_Intro|EC2]] instances used for both staging and target environments.
- **Storage Costs:** [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] storage costs for staged data, [[00_Master_Links_and_Intro|EBS]] volumes for target instances.
- **Data Transfer Costs:** Network bandwidth usage for initial transfer and ongoing replication.

**Optimization Strategies:**
- Use [[Reserved Instances (RIs)]] or Savings Plans to reduce compute costs.
- Optimize instance types based on workload needs to avoid over-provisioning.
- Enable data compression and use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] lifecycle [[policies]] to manage storage costs effectively.

#### 7. COMPETITOR COMPARISON

**Similar Services:**
- **VMware HCX:** Provides similar functionality but is more suited for VMware environments, offering advanced network mapping capabilities.
- **Microsoft Azure Migrate:** Offers assessment and migration services from on-premises to [[Azure]], with a focus on hybrid cloud scenarios.

**Architectural Trade-offs:**
- AWS MGN offers tighter integration with the broader AWS ecosystem (e.g., [[00_Master_Links_and_Intro|EC2]], [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]), whereas VMware HCX is more aligned with VMware infrastructure.
- Azure Migrate requires migrating workloads to Azure, which may not align with multi-cloud strategies.

#### 8. INTEGRATION

**[[cloudwatch]]:**
- [[MGN]] integrates with [[cloudwatch]] for [[vpc-flow-logs|logging]] and monitoring purposes, allowing you to track replication metrics and receive alerts on migration status.

**[[00_Master_Links_and_Intro|IAM]]:**
- [[00_Master_Links_and_Intro|IAM]] roles are used to grant necessary permissions to the replication agent and MGN service actions.
  
**[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:**
- MGN leverages VPCs for secure communication between source and target environments. Integration with [[appsync|security]] groups ensures network isolation.

#### 9. USE CASES

**Enterprise Examples:**
- **Data Center Migration:** Migrating a large data center from on-premises to AWS, ensuring minimal downtime.
- **Lift-and-Shift Workloads:** Moving legacy applications to the cloud without significant changes, maintaining existing configurations and dependencies.
- **[[00_Master_Links_and_Intro|Disaster Recovery]]:** Setting up real-time replication for [[00_Master_Links_and_Intro|disaster recovery]] scenarios with automatic failover capabilities.

#### 10. DIAGRAMS

**Placeholder Diagram:**
```
Source Servers (On-Prem) --> MGN Replication Agent --> S3 Staging Area --> EC2 Target Instances
                                    |                                      |
                                    v                                      v
                                VPC Configuration                       Network Security Groups
```

#### 11. EXAM TRAPS

> [!danger] Distractor
- **Wrong Use Case: Data Replication Without Stateful Workloads**: MGN is designed primarily to migrate stateful applications and workloads, such as databases and servers with persistent data. It may not be the right choice for replicating static or dynamic data without a stateful context.
- **Incompatible Workload Types**: Applications that do not require full migration (e.g., containerized services already running in AWS) might find MGN overkill. Tools like [[Amazon ECS]] or [[eks]] are more appropriate for container orchestration and deployment.
- **Non-Virtual Machine Environments**: MGN is optimized for migrating VMs, so it may not be the best solution if your application environment consists of bare-metal servers without a virtualization layer.
- **Complex Custom Integration Needs**: For applications requiring extensive custom scripting or complex integration with third-party tools during migration, MGN might not provide sufficient flexibility compared to more customizable solutions like [[AWS CloudFormation]].
- **Low-Bandwidth Networks**: While MGN supports WAN acceleration, environments with very low bandwidth connections may experience significant latency and inefficiency.

#### 12. DECISION MATRIX

| Use Case                      | Recommended Solution                    |
|-------------------------------|----------------------------------------|
| Initial migration of large datasets   | Full Replication                     |
| Ongoing real-time data synchronization| Incremental Replication              |
| Configuring target environment       | Custom Post-Migration Scripts        |

#### 13. EXAM SCENARIOS

> [!abstract] Exam Tip
- **Scenario Example:** A company needs to migrate its on-premises infrastructure to AWS without significant downtime. Which service would be most appropriate, and what steps are required?
- **Answer:** MGN is the appropriate service. Steps include installing the replication agent, configuring source servers, initiating full and incremental data transfer, launching target instances, and monitoring progress.

#### 14. TIER-3 TROUBLESHOOTING

> [!warning] Quota
1. **[[kms]] Key Policy Conflicts**: Ensure that the [[kms]] key used by MGN has appropriate [[policies]] set up, as incorrect permissions can lead to encryption/decryption failures during migration.
2. **Resource Quota Exceedances**: Monitor and adjust resource quotas for services like [[ec2]] and [[00_Master_Links_and_Intro|EBS]], which are critical for MGN operations. Exceeding these limits could halt the migration process.
3. **Agent Installation Failures**: Check agent installation logs on source servers for [[api-gateway|errors]] related to OS compatibility or permissions issues that might prevent successful setup of the replication process.

#### 15. RECENT UPDATES

- **GA Features (2026)**: Monitor announcements for any new general availability features from AWS that enhance MGN functionality, such as improved support for hybrid cloud environments.
- **Deprecated Items**: Keep an eye on the [[AWS documentation]] for any services or functionalities related to MGN that may be marked for deprecation by 2026. Adjust your migration strategies accordingly.

By following these enhanced guidelines and ensuring comprehensive governance and troubleshooting steps are in place, you can leverage AWS Application Migration Service (MGN) effectively while avoiding common pitfalls and staying up-to-date with the latest advancements.