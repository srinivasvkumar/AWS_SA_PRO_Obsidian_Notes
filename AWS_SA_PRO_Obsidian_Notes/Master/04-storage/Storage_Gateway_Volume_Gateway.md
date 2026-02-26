### Advanced Architecture

At its core, the [[Storage Gateway]] [[storage-gateway|Volume Gateway]] is a hybrid cloud storage solution that allows you to seamlessly integrate on-premises applications with cloud-based storage. It provides three storage modes: Cached Volumes, Stored Volumes, and [[storage-gateway|Volume Gateway]] as a Service (VGWaaS). The focus here will be on Cached and Stored Volumes.

#### Cached Volumes

In Cached Volumes mode, the gateway maintains a local cache of frequently accessed data while storing the entire data set in Amazon Elastic Block Store ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]) snapshots. This configuration enables low-latency access to frequently used data while providing durable, offsite backups.

![Mermaid Code](mermaidcode://?v=1&mode=sequence&p=sequenceDiagram|Storage%20Gateway--%3EVolume%20Gateway:%20Read%20Request&Storage%20Gateway--%3ECached%20Volume%20(Local%20SSD):%20Read%20from%20Cache&Cache%20Volume%20(Local%20SSD)--%3EStorage%20Gateway:%20Response&Storage%20Gateway--%3EAmazon%20EBS:%20Snapshot%20Data%20Fetching)

#### Stored Volumes

Stored Volumes mode stores all data locally and then asynchronously backs up point-in-time copies to Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots. With Stored Volumes, your primary data resides within the [[storage-gateway|volume gateway]] and offers a lower-cost option compared to Cached Volumes.

![Mermaid Code](mermaidcode://?v=1&mode=sequence&p=sequenceDiagram|Storage%20Gateway--%3EVolume%20Gateway:%20Write%20Request&Volume%20Gateway--%3ELocal%20HDD:%20Store%20Primary%20Data&Volume%20Gateway--%3EAmazon%20EBS:%20Backup%20to%20Snapshots)

### Comparison & Anti-Patterns

| Service | Use Case |
|---|---|
| [[Storage Gateway]] [[storage-gateway|Volume Gateway]] | High-performance, low-latency access to frequently accessed data using Cached Volumes.<br>Lower-cost infrequent access to primary data using Stored Volumes. |
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] | Direct block-level storage integration with [[ec2]] instances.<br>Not suitable for hybrid cloud scenarios without additional software layers. |
| Amazon [[fsx]] for Windows File Server | Centralized storage across multiple [[ec2]] instances.<br>Requires managing file servers and protocols. |

Anti-patterns include using [[Storage Gateway]] [[storage-gateway|Volume Gateway]] for backup and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes when other services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] or [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] would provide more specialized functionality.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can enforce granular control over access to specific resources and actions. For example, restricting iSCSI initiators to specific IP addresses:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "storagegateway:CreateVolume"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:SourceIp": [
                        "192.168.1.0/24"
                    ]
                }
            }
        }
    ]
}
```

Cross-account access involves creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account, granting permissions to the target account via bucket [[policies]], and assuming the role from the target account. Additionally, Service Control [[policies]] (SCPs) at the organization level can centralize governance across multiple accounts.

### Performance & Reliability

Throttling limits depend on the type of gateway, ranging from 200 requests per second for Cached Volumes to 100 requests per second for Stored Volumes. To manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies using the AWS SDK or third-party libraries.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], deploy multiple gateways across different Availability Zones and ensure they have shared access to the same [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls involve monitoring usage trends through [[cloudwatch|CloudWatch alarms]] and setting [[billing]] alerts based on thresholds. Calculation examples include comparing costs between Cached and Stored Volumes, considering factors such as storage capacity, read/write operations, and snapshot storage.

### Professional Exam Scenarios

**Scenario 1**

*Objective*: Design a highly available architecture for a web application requiring low-latency access to frequently changing data.

*Correct answer*: Implement a [[Storage Gateway]] Cached Volumes configuration that stores the majority of data in Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots while maintaining a local cache for performance. Deploy the gateway across at least two Availability Zones to achieve high availability.

*Incorrect answer*: Use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] directly attached to [[ec2]] instances since it does not offer hybrid cloud capabilities.

**Scenario 2**

*Objective*: Enforce strict [[appsync|security]] [[policies]] governing access to a [[Storage Gateway]] [[storage-gateway|Volume Gateway]].

*Correct answer*: Create custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that restrict specific actions (e.g., `CreateVolume`) to authorized IP addresses. Implement cross-account access by creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and assuming it from the target account. Optionally, apply Service Control [[policies]] (SCPs) at the organization level to centralize governance.

*Incorrect answer*: Implement coarse-grained [[appsync|security]] [[policies]] allowing unrestricted access to the [[Storage Gateway]] [[storage-gateway|Volume Gateway]].