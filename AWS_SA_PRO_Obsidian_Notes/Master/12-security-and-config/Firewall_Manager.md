**[[RDS_Instance_Types|1. Advanced Architecture]]**

Firewall Manager operates at the enforcement point of [[network-and-dns-firewall|AWS Network Firewall]], providing centralized management and visibility across accounts and applications. It utilizes two key components: Managed Rules and Rule Groups. Managed Rules are pre-configured rules provided by AWS, while Rule Groups are collections of rules that can be applied to one or more firewalls.

Firewall Manager supports tag-based rule application, enabling you to apply rules consistently across resources with specific tags. This allows for granular control over which resources are protected by certain rules.

Global Considerations:

- Firewall Manager is regional-specific and supports multiple regions within an account. However, it doesn't have global coverage like [[shield|AWS Shield]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver.
- Firewall Manager integrates with [[organizations|AWS Organizations]] allowing you to manage and enforce rules across member accounts.

Under The Hood Mechanics:

- Firewall Manager uses [[lambda|AWS Lambda]] to perform configuration checks and modifications in real-time.
- It leverages [[step-functions|AWS Step Functions]] for workflow coordination when applying changes across multiple accounts.
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] [[cloudformation|StackSets]] are used to create and manage consistent configurations across different accounts and regions.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
|---|---|
| Firewall Manager | Centralized management of Network Firewall rules. Ideal for managing multiple accounts and ensuring consistent [[appsync|security]] posture. |
| [[appsync|Security]] Groups | Stateless, stateful, and application-level protection. Not suitable for centralized management as they must be set per resource. |
| Network ACLs | Low-level network traffic filtering. Difficult to manage centrally due to their low-level nature. |

Anti-patterns:

- Using Firewall Manager as a replacement for [[appsync|Security]] Groups or Network ACLs: These services offer different features and should be used together for optimal [[appsync|security]].
- Applying Firewall Manager rules without understanding the impact: Always test new rules in a staging environment before deploying them to production.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] JSON Snippet Example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "firewallmanager:Describe*",
              "firewallmanager:Get*",
              "firewallmanager:List*"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-Account Access:

- Enable cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles assuming necessary permissions.
- Configure [[organizations]] to automatically invite new member accounts.

Organization SCPs:

- Implement Organization Service Control [[policies]] (SCPs) to restrict actions on supported services.
- Ensure SCPs don't interfere with Firewall Manager functionality.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:

- Firewall Manager has a request limit of 5 requests per second.
- Exponential backoff strategies should be implemented if this limit is reached.

HA/DR Patterns:

- Firewall Manager supports multiple regions, so enable multi-region setup for high availability.
- For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], ensure backups are taken regularly and stored in a separate region.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular Cost Controls:

- Utilize tagging to track costs associated with Firewall Manager rules.
- Monitor usage through AWS [[billing|Cost Explorer]] and implement budget alerts.

Calculation Examples:

- Calculate costs based on the number of firewalls managed, regions utilized, and data processed.
- Refer to the [AWS Pricing page](https://aws.amazon.com/network-firewall/pricing/) for up-to-date pricing information.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
Your organization has 50 AWS accounts distributed across three regions. You want to implement a centralized [[appsync|security]] policy for all VPCs using Network Firewall. Which solution meets these requirements?

Correct Answer: Implement Firewall Manager with [[organizations|AWS Organizations]] and apply the required Network Firewall rules centrally.

Incorrect Answers:

- Using [[appsync|Security]] Groups or Network ACLs alone would not provide centralized management capabilities.
- Configuring individual Network Firewalls in each account without centralized management could lead to inconsistent configurations.

Scenario 2:
You need to implement a [[appsync|security]] solution that supports both Network Firewall and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] integration. What service should you use?

Correct Answer: Firewall Manager

Incorrect Answers:

- [[appsync|Security]] Groups do not support Application Load Balancer integration directly.
- Network ACLs lack [[ALB]] integration and require manual configuration per resource.