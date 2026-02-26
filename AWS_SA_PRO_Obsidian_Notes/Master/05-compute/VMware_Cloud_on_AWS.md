**Advanced Architecture**

VMware Cloud on AWS (VMC) is a jointly engineered solution by VMware and AWS that delivers a consistent hybrid cloud experience by extending on-premises vSphere environments to the AWS global infrastructure. VMC uses dedicated AWS Nitro System-based hardware along with VMware Cloud Foundation (vCF) stack components: vSAN, NSX-T, and vCenter Server. The architecture comprises the following key elements:

* **SDDC Grouping**: Multiple logical networks, IP address blocks, and firewall rules are organized within SDDC groups.
* **Hybrid Connectivity**: Leverages [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] to connect on-premises data centers with VMC.
* **Stretched Cluster**: Provides workload mobility across availability zones while maintaining sub-millisecond latency.
* **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]]**: VMware Site Recovery Manager (SRM) enables [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] between on-premises and VMC or between two VMC instances.

[[RDS_Instance_Types|Global scale considerations]] in VMC include:

* **Region Availability**: VMC is available in selected AWS regions only.
* **Resource Allocation**: Each SDDC instance has its own resource pool, which can span multiple hosts.
* **Networking**: Network segments can be stretched across multiple availability zones using NSX-T.
* **Backup and [[dr]]**: VMware vSphere Replication, vSAN, and SRM enable backup and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

**Comparison & Anti-Patterns**

| Service | Comparison Criteria | VMware Cloud on AWS | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/05-compute/others|AWS Outposts]] | Amazon [[ec2]] |
|---|---|---|---|---|
| VMware Cloud on AWS | Management Stack | VMware vCF | VMware vCF Lite | [[AWS Systems Manager]] |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/05-compute/others|AWS Outposts]] | Physical Hardware | VMware provided | AWS provided | Customer provided |
| Amazon [[ec2]] | Scalability | Limited by SDDC size | Near-infinite elasticity | Near-infinite elasticity |

Common anti-patterns include:

* Running VMC as a single account without proper Organization [[appsync|Security]] [[policies]].
* Not monitoring throttling limits and applying exponential backoff strategies.
* Ignoring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] [[iam|best practices]] such as granular cost controls.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting cross-account access to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] resources from other AWS accounts. An example JSON policy includes:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```

Cross-account access involves attaching an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to the VMC account with permissions granted to another AWS account. Additionally, Service Control [[policies]] (SCPs) at the organization level ensure [[appsync|security]] and governance. For instance, enforcing encryption requirements:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "NotAction": "ec2:DescribeVolumes",
            "Resource": "*",
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:TagValue": "yes"
                },
                "Null": {
                    "aws:RequestTag/Encrypted": "true"
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Throttling limits apply to several operations in VMC, including adding hosts, creating new SDDC instances, and deleting SDDC instances. Exponential backoff strategies should be applied when these limits are exceeded. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include:

* Stretched clusters for low-latency workload mobility.
* VMware SRM for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] between on-premises and VMC or between two VMC instances.
* NSX-T for load balancing, microsegmentation, and network automation.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls involve setting up [[billing]] alarms based on usage trends and reserving capacity. Calculating costs requires understanding the underlying pricing model:

* Host configuration: $0.10 per hour (prorated per host).
* Data transfer: $0.09 per GB (inbound) and $0.19 per GB (outbound).
* Additional services: Additional fees for additional services like Elastic vGPU, vSAN, and NSX-T.

Example calculations:

* One host running for one month would generate approximately $720 in charges ($0.10 * 24 hours * 30 days * 1 host).
* Transferring 1 TB of outbound data would result in around $190 in charges (1 TB \* 1024 GB \* $0.19 per GB).

**Professional Exam Scenarios**

Scenario 1: A company wants to implement a hybrid cloud environment using VMware Cloud on AWS. They have an existing vSphere environment and want to extend it into AWS. Which of the following solutions provides the desired functionality?

A) Implementing a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connection between the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and the on-premises vSphere environment.
B) Setting up a Direct Connect link between the on-premises vSphere environment and VMC.
C) Using an internet gateway to connect the on-premises vSphere environment with VMC.

Correct answer: B. Direct Connect allows secure, reliable connectivity between the on-premises vSphere environment and VMC over a private connection. Incorrect answers do not provide the required integration.

Scenario 2: A customer needs to set up a highly available VMware Cloud on AWS environment. What steps should they take to achieve high availability?

A) Set up a stretched cluster spanning multiple hosts in different VMC zones.
B) Enable Elastic vGPU on all hosts in the VMC environment.
C) Configure VMware SRM for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] between on-premises and VMC.

Correct answer: A. Stretched clusters maintain sub-millisecond latency between hosts, providing higher availability than non-stretched clusters. Incorrect answers do not directly contribute to high availability. However, combining them with stretched clusters might provide additional benefits.