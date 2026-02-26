**Advanced Architecture**

At its core, [[vm-migration|Server Migration Service (SMS)]] is an agentless service that helps migrate physical, VMware, and cloud-based servers into AWS. The SMS agent collects server metadata, including configuration, performance, and data, and sends it to SMS for analysis. This allows for automated replication of servers in the source environment to Amazon [[ec2]] instances.

The SMS architecture consists of several components:

1. **SMS Components**: Includes the SMS console, API, and agent responsible for collecting metadata from source servers.
2. **[[AWS Application Discovery Service]] Connector**: A component used to discover applications running on VMware vSphere and Microsoft Hyper-V environments.
3. **[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]]**: A service used for transferring data to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[fsx]] for Windows File Server.

*[[RDS_Instance_Types|Global Scale Considerations]] and Under the Hood Mechanics*

SMS supports multiple regions and accounts, allowing you to manage migrations across different environments. To ensure efficient metadata collection and transfer, SMS uses intelligent polling, which reduces network load by only sending updated metadata. Additionally, SMS can handle large-scale migrations using incremental replication, ensuring minimal impact on the production environment.

**Comparison & Anti-Patterns**

| Service | Use Cases |
| --- | --- |
| [[vm-migration|Server Migration Service (SMS)]] | Physical, VMware, and cloud-based servers migration. |
| [[AWS Application Discovery Service]] | Pre-migration application discovery and dependency mapping. |
| AWS Database Migration Service ([[dms]]) | Homogeneous and heterogeneous database migrations. |
| AWS Server Migration Connector (SMC) | Agentless migration for VMware vSphere and Microsoft Hyper-V. |

Anti-patterns include using SMS as a continuous replication solution instead of one-time or periodic migrations. SMS does not support ongoing synchronization like AWS [[dms]].

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sms:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-Account Access:

To enable cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the destination account with permissions for SMS actions and attach a trust policy for the source account.

Organization SCPs:

Use Organizational Service Control [[policies]] (SCPs) to limit SMS usage within your organization. For example, restrict specific users or groups from creating new replications or managing existing ones.

**Performance & Reliability**

Throttling Limits:

SMS has a default throttle limit of 100 tasks per AWS account. If you reach this limit, implement exponential backoff strategies when making API calls.

High Availability/Disaster Recovery Patterns:

Implement active-passive or pilot light [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns using SMS. For instance, maintain a duplicate environment in another region and periodically update it using replication jobs.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:

Control costs by setting up [[billing]] alarms based on usage metrics and terminating unused replications. Calculate costs using the following formula:

Total Cost = Number of Replications \* Cost per Replication + Data Transfer Charges

**Professional Exam Scenarios**

Scenario 1:

An organization wants to migrate their physical servers to AWS. They have 500 servers with varying configurations. Which approach should they follow?

*Option 1*: Use SMS to automate the process of migrating all 500 servers to Amazon [[ec2]].

*Option 2*: Manually configure each server and migrate them to Amazon [[ec2]].

Correct Answer: Option 1. Using SMS simplifies the migration process by automating metadata collection and enabling incremental replication.

Incorrect Answer: Option 2. Manual configuration is time-consuming, error-prone, and lacks the benefits of SMS automation.

Scenario 2:

An organization wants to migrate their VMware vSphere environment to AWS. They want to minimize disruption during the migration process. Which approach should they follow?

*Option 1*: Use SMS to replicate VMs to Amazon [[ec2]] while maintaining production availability.

*Option 2*: Use the VMware Cloud Foundation to set up a hybrid cloud environment and manually migrate workloads.

Correct Answer: Option 1. SMS provides agentless migration capabilities, reducing the need for manual intervention and minimizing disruptions.

Incorrect Answer: Option 2. While VMware Cloud Foundation enables hybrid cloud scenarios, it may introduce additional complexity and cost compared to using SMS.