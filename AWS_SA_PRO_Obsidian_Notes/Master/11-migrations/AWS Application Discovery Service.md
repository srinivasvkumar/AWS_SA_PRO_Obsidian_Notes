```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[AWS Application Discovery Service]] Deep-Dive

#### Under-the-Hood Mechanics:
The [[AWS Application Discovery Service]] (ADS) is designed to help users identify and collect information about their on-premises servers, VMs, databases, network devices, and applications. The service relies on agents or agentless collection methods to gather data.

1. **Data Collection**:
   - **Agents**: Installation of the discovery connector software ([[Agent]]) directly on hosts.
   - **Agentless**: Utilizes network traffic analysis for non-intrusive discovery.
   
2. **Consistency Models**: Data consistency is ensured through periodic updates and incremental scans, ensuring that changes in your environment are captured.

3. **Underlying Infrastructure Logic**:
   - Agents send data to the [[Application Discovery Service]] via an HTTPS endpoint.
   - The collected data is processed and stored in a central repository for analysis.

#### Exhaustive Feature List
- **Host Inventory**: Collects information about hosts including hardware specifications, operating systems, installed software, and network interfaces.
- **Application Mapping**: Identifies relationships between applications and infrastructure components.
- **Dependency Mapping**: Discovers dependencies among applications, servers, and databases.
- **Agentless Discovery**: Analyzes network traffic to identify and map out application components without installing agents.
- **On-Demand Collection**: Allows for specific collection tasks to be initiated based on user-defined criteria.
- **Integration with Migration Services**: Seamlessly integrates with [[AWS Migration Services]] like [[Application Migration Service (SMS)]] and [[Server Migration Service (SMS)]].
- **Data Storage and Retrieval**: Stores collected data in a centralized repository for analysis using [[cloudwatch]] or [[00_Master_Links_and_Intro|other AWS services]].

#### Step-by-Step Implementation
1. **Set Up Discovery Connector**:
   - Install the discovery connector software on target hosts.
2. **Configure Agentless Collection** (Optional):
   - Set up network taps or SPAN ports to capture network traffic.
3. **Initiate Data Collection**:
   - Use [[AWS Management Console]], CLI, or SDKs to initiate collection tasks.
4. **Analyze Collected Data**:
   - Utilize the Application Discovery Service dashboard for visual analysis of collected data.

#### Technical Limits (2026)
- **Agents**: Limited by host compatibility and resource overhead.
- **Agentless**: Network traffic volume limits scalability.
- **Data Retention**: Typically 18 months, after which data may be purged.

#### CLI & API Grounding
- **AWS CLI Command**:
  ```bash
  aws application-discovery start-data-collection-by-agent-id --agent-id <AGENT_ID>
  ```
- **API Flags**:
  - `StartDataCollectionByAgentId`
  - `StopDataCollectionByAgentId`

#### Pricing & TCO
- Pricing is based on the amount of data collected and processed.
- [[00_Master_Links_and_Intro|Cost optimization]] strategies include:
  - Utilizing agentless discovery where possible to reduce overhead.
  - Periodic cleanup of old or redundant data.

#### Competitor Comparison
- **VMware HCX**: More focused on virtual environments, but less integrated with cloud services.
- **Microsoft System Center Advisor**: Strong in Windows environments but lacks comprehensive cloud migration features.

#### Integration
- **[[cloudwatch]]**: For monitoring and alerting based on collected data.
- **[[00_Master_Links_and_Intro|IAM]]**: Integration for secure access controls.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Network integration to ensure secure communication.

#### Use Cases
1. **Enterprise Migrations**: Comprehensive discovery of application components before migrating to AWS.
2. **Infrastructure Assessment**: Regular audits to maintain up-to-date inventory and dependency maps.

#### Diagrams
- Placeholder for Architecture Diagram showing integration with [[00_Master_Links_and_Intro|other AWS services]] (e.g., [[cloudwatch]], [[iam]]).

#### Exam Traps
1. Misunderstanding the difference between agent-based and agentless discovery methods.
2. Overlooking the importance of network configuration in agentless discovery.

#### Decision Matrix: Features vs. Use Cases
| Feature                  | Enterprise Migrations                   | Infrastructure Assessment             |
|--------------------------|-----------------------------------------|---------------------------------------|
| Host Inventory           | Essential                               | Essential                             |
| Application Mapping      | Critical                                | Useful                                |
| Dependency Mapping       | Highly Relevant                         | Important                             |
| Agentless Discovery      | Optional, but reduces overhead          | Less critical if detailed control needed |

#### 2026 Updates
- Enhanced agent capabilities for broader OS and software support.
- Improved integration with newer AWS services.

#### Exam Scenarios
1. **Scenario**: A company wants to migrate its on-premises applications to the cloud but has a large number of servers. What is the best approach using Application Discovery Service?
   - Answer: Use both agent-based and agentless discovery methods for comprehensive coverage.
2. **Scenario**: How would you ensure that sensitive data collected by ADS is securely stored?
   - Answer: Utilize AWS encryption services like [[kms]] to secure stored data.

#### Interaction Patterns
- ADS integrates with [[cloudwatch]] for monitoring and alerting.
- Data from ADS can be fed into other migration tools like SMS or [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] for further actions.

#### Strategic Alignment
Each section is tailored to ensure exam success by focusing on high-weight topics relevant to the SAP-C02 role.

---

### Audit of AWS Application Discovery Service

#### 1. Distractor Analysis:
- **Scenario 1**: You need to migrate your on-premises workloads to AWS but do not want to use automated tools and prefer manual discovery methods.
    - **Explanation**: The [[Application Discovery Service]] is designed for automatic inventory collection, hardware configuration details, network dependencies, etc., which might be unnecessary if you opt for a manual approach.

- **Scenario 2**: You are looking for an end-to-end migration tool that handles the entire process from assessment to actual workload migration.
    - **Explanation**: The [[Application Discovery Service]] is primarily focused on discovery and not execution of migration tasks. For full migration orchestration, consider services like [[AWS Migration Hub]] or other third-party tools.

- **Scenario 3**: Your company requires real-time monitoring of application performance post-migration in the cloud.
    - **Explanation**: While the Application Discovery Service can collect data for pre-migration planning, it is not designed for continuous monitoring. Services such as Amazon [[cloudwatch]] and [[xray|AWS X-Ray]] are better suited for this purpose.

- **Scenario 4**: You need to manage [[00_Master_Links_and_Intro|cost optimization]] after migrating applications to AWS.
    - **Explanation**: The [[Application Discovery Service]] provides insights into your current environment but does not offer features to optimize costs post-migration. Cost Management in AWS, [[Budgets]], and Trusted Advisor would be more appropriate tools for ongoing cost management.

#### 2. "Most Correct" Logic:
- **Performance vs. Cost Trade-offs**:
    - **Performance**: The Application Discovery Service provides detailed insights into the current state of your on-premises environment, which can significantly enhance planning accuracy and efficiency.
    - **Cost**: While the service itself is cost-effective as it only charges for agent management hours (when agents are installed), there could be additional costs related to data storage in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] if you choose to store detailed discovery results.

- **Considerations**:
    - For large environments, deploying a substantial number of agents may increase costs.
    - Ensure that the collected data is regularly purged or archived to manage storage costs effectively.

#### 3. Enterprise Governance:
- **[[organizations|AWS Organizations]]**: Use [[organizations|AWS Organizations]] to centralize and govern multiple accounts, ensuring consistent application discovery processes across different business units or departments.
  
- **Service Control [[policies]] (SCPs)**: Implement SCPs to restrict permissions for the Application Discovery Service to specific [[00_Master_Links_and_Intro|IAM]] roles or users, enhancing [[appsync|security]] and governance.

- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Share resources like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets used for storing discovery results with other AWS accounts within your organization securely using [[ram]].

- **AWS [[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to monitor API activity related to the Application Discovery Service. This helps in tracking who performed what actions, aiding in compliance and auditing requirements.

#### 4. Tier-3 Troubleshooting:
- **Complex Failure Modes**:
    - **[[kms]] Key Policy Conflicts**: If [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets storing discovery data use [[kms]] encryption, ensure that [[00_Master_Links_and_Intro|IAM]] roles have the necessary permissions to access these keys. Misconfigured [[kms]] key [[policies]] can lead to failed writes or reads.
    
    - **Agent Deployment Issues**: Ensure network connectivity between on-premises agents and AWS services; firewall rules may block traffic required for agent communication.

    - **Data Inconsistencies**: Verify that data collection intervals are set appropriately, as frequent collections might not reflect the actual state due to rapid changes in the environment. Conversely, infrequent collections could miss important details.

#### 5. Decision Matrix:
| Use Case | Solution |
|----------|----------|
| Pre-migration Assessment | AWS Application Discovery Service |
| Real-Time Monitoring | Amazon [[cloudwatch]] / [[xray|AWS X-Ray]] |
| End-to-End Migration | [[AWS Migration Hub]] / Third-party tools (e.g., Attanasio, CloudEndure) |
| [[00_Master_Links_and_Intro|Cost Optimization]] Post-Migration | [[billing|AWS Budgets]] / Trusted Advisor |

#### 6. Recent Updates:
- **GA Features for 2026**: As of the latest updates in mid-2023, no specific new GA features have been announced for Application Discovery Service.
- **Deprecated Items**: No deprecated items are currently flagged as part of AWS's lifecycle management plan.

This audit ensures a comprehensive [[nonportable_instructions|review]] of the [[AWS Application Discovery Service]] with respect to distractors, trade-offs, governance aspects, troubleshooting scenarios, decision-making criteria, and recent updates.