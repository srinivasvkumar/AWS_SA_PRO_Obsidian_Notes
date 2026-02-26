## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, an Elastic IP Address (EIP) is a static, public IPv4 address designed for dynamic cloud computing. An EIP can be associated with any instance in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] within the same region. Once associated, the instance is assigned a private IPv4 address from the IPv4 address range of your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

### Global Considerations

EIPs are regional resources, but AWS provides a mechanism to associate them with [[ec2]] instances in a different region. This is achieved using the "[[ec2]] Global IP" feature, which enables associating EIPs with instances across different regions by creating a global association. However, this comes at additional costs and increases latency compared to local associations.

### Under the Hood Mechanics

When you allocate an EIP, AWS reserves it for your use and bills you for it regardless of whether it is attached to an instance or not. To minimize costs, unassociated EIPs should be released. The [[billing]] rate depends on the region and is prorated per hour. When an EIP is associated with a network interface, the charges apply only to the time when the instance is running.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service          | Use Case                                                                   |
| ---------------- | ------------------------------------------------------------------------- |
| **Elastic IP**   | Persistent, static public IP addressing for [[ec2]] instances                |
| **NLB/ALB**      | Application load balancing, OSI layer 7 routing                           |
| **VPN/Direct Connect** | Hybrid connectivity between on-premises networks and AWS VPCs       |
| **[[Transit gateway]]** | Multi-region connectivity, large-scale network hub-and-spoke topologies |

### Common Design Mistakes

- Using EIPs as a primary means for communication between instances instead of utilizing services like [[ALB]], [[NLB]], or ENIs.
- Forgetting to release unassociated EIPs, leading to unnecessary costs.
- Ignoring the increased latency and additional costs associated with global EIP associations.

## [[RDS_Instance_Types|3. Security & Governance]]

To control access to EIPs, implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON statements. Here's an example that grants access to create, associate, and dissociate EIPs for specific users:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "ec2:AssociateAddress",
              "ec2:AllocateAddress",
              "ec2:DescribeAddresses"
            ],
            "Resource": [
              "*"
            ],
            "Condition": {
              "StringEquals": {
                "ec2:Region": [
                  "us-west-2"
                ]
              }
            },
            "Effect": "Deny",
            "Action": [
              "ec2:ReleaseAddress"
            ],
            "Resource": [
              "*"
            ],
            "Condition": {
              "StringEquals": {
                "ec2:ResourceTag/Name": [
                  "Unallocated_EIP"
                ]
              }
            }
          }
    ]
}
```

Cross-account EIP access can be enabled through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and resource-based [[policies]]. Additionally, [[organizations]] can enforce [[appsync|security]] [[policies]] using Service Control [[policies]] (SCPs):

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyEIPAllocationExceptAllowed",
      "Effect": "Deny",
      "Action": [
        "ec2:AllocateAddress"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringNotEqualsIfExists": {
          "ec2:Region": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

EIPs do not have dedicated throttling limits, but there are general [[ec2]] API limits related to EIP operations. If you encounter [[api-gateway|errors]] due to reaching these limits, follow AWS recommended practices for handling throttled requests, including exponential backoff strategies.

HA/DR patterns involving EIPs may require manual intervention during failover scenarios. Implement automated recovery mechanisms using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], Systems Manager Automation, or other third-party tools.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include monitoring usage trends, setting up alerts for unusual activity, and releasing unused EIPs. Calculate EIP costs using the following formula:

```
Cost = (Hours used * Rate per hour) + (Monthly charge if unassociated)
```

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario A

Suppose you need to provide high availability for a web application deployed in multiple regions, and each instance must have a static, globally accessible IP address. Which approach would meet these requirements while minimizing costs?

#### Correct Answer

Use Elastic IPs in combination with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover. Allocate an EIP for each instance, and configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] records to monitor their health. In case of failure, traffic will automatically be redirected to healthy instances.

#### Incorrect Answers

- Using EIPs without [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover: While this approach meets the requirement for a static IP address, it fails to ensure high availability.
- Using ALBs without EIPs: Although ALBs offer high availability and load balancing capabilities, they don't inherently provide a single, globally accessible static IP address.

### Scenario B

Consider a scenario where a company has several AWS accounts organized under an [[AWS Organization]]. They want to restrict EIP allocation to specific AWS Regions. How could this be accomplished using Service Control [[policies]] (SCPs)?

#### Correct Answer

Create an [[SCP]] that denies `ec2:AllocateAddress` action outside specified regions:

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyEIPAllocationOutsideRegions",
      "Effect": "Deny",
      "Action": [
        "ec2:AllocateAddress"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringNotEqualsIfExists": {
          "ec2:Region": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}
```

Attach this [[SCP]] to the desired organizational unit (OU) containing the relevant AWS accounts.

#### Incorrect Answers

- Creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy for individual user(s) or role(s): This method does not effectively address the requirement for organization-wide restrictions.
- Applying the [[SCP]] at the root level: This would deny EIP allocation for all child accounts, potentially impacting unintended accounts.