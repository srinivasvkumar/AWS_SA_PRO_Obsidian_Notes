## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, Outposts servers are physical infrastructure deployments of AWS in customers' data centers or co-location spaces. They provide a consistent set of AWS services, APIs, and tools between on-premises and cloud environments. An Outpost is composed of one or more server racks that can be configured with various instance types and volumes.

Outposts support several deployment options:

- **Hosted Outposts**: AWS manages the infrastructure, but you have full control over the instances and storage.
- **Local Gateway mode**: Connect your on-premises network to an Outpost using a [[Transit gateway]] or virtual private cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) without requiring dedicated [[ec2]] instances.
- **Native Outposts**: You manage the entire infrastructure, including instances and storage.

Global services such as Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]], Amazon [[cloudwatch]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] [[cloudformation|StackSets]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS License Manager]] extend native capabilities to hybrid environments.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service          | Use Case                                                              |
| ---------------- | -------------------------------------------------------------------- |
| Outposts        | Low latency, local data processing, and specific regulatory needs   |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/05-compute/others|AWS Wavelength]]  | Mobile edge computing and ultra-low latency applications             |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpc|AWS Local Zones]]  | Regional workloads requiring single-digit millisecond latencies      |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Snow Family]]  | Disconnected or extremely low bandwidth locations                    |

Anti-pattern: Using Outposts when performance requirements do not justify the added complexity and cost compared to running the same workload fully in the cloud.

## [[RDS_Instance_Types|3. Security & Governance]]

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "outposts:Describe*, *-outposts*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Cross-Account Access

Use a role with permissions to create and manage Outposts resources within the account. Then, attach a trust policy allowing the management account to assume the role.

### Organization Service Control [[policies]] (SCPs)

[[organizations]] SCPs limit the actions available at the member account level. For example, restricting `outposts:CreateOutpost` prevents creating new Outposts.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the instance type used in the Outpost. Refer to [the documentation](https://docs.aws.amazon.com/outposts/latest/userguide/quotas.html) for details. Implement exponential backoff strategies when encountering throttled requests.

For high availability, distribute instances across multiple racks and ensure connectivity to the primary region via a [[Transit gateway]] or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]. To improve [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], synchronize data between the Outpost and AWS Regions using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or similar tools.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include setting up [[Budgets]] per Outpost, monitoring usage with Amazon [[cloudwatch]], and automating scaling based on custom metrics. Calculate costs by considering:

- Instance types and their associated costs
- Storage configuration and pricing
- Networking charges (data transfer, NAT gateways, etc.)

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario A

You are designing a hybrid solution for a retail company that requires real-time inventory updates. The system must handle peak traffic during holiday seasons.

Correct Answer: Outposts
Justification: Due to the need for low latency and handling peak traffic, Outposts would offer better performance than alternatives like [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Wavelength]] or Local Zones.

Incorrect Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Wavelength]]
Justification: Although [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Wavelength]] targets mobile edge computing and ultra-low latency applications, it does not meet the requirement for handling peak traffic during holiday seasons.

### Scenario B

A financial institution requires strict governance and [[appsync|security]] measures for their hybrid environment. They want to enforce tight control over which users can create and manage Outposts resources.

Correct Answer: [[organizations]] SCPs and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]
Justification: By implementing [[organizations]] SCPs, the financial institution could limit actions available at the member account level. Simultaneously, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] allow fine-grained control over who can perform specific tasks related to Outposts resources.

Incorrect Answer: Using separate accounts for each Outpost
Justification: While separating accounts might provide some isolation, it does not address the need for centralized control and enforcement of [[appsync|security]] measures.