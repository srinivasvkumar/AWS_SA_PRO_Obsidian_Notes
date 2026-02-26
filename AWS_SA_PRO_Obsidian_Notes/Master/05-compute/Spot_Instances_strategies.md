### Advanced Architecture
At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] is an auction-based model where [[ec2]] instances are sold at a discounted price compared to On-Demand prices. The supply and pricing of Spot Inststances are managed by AWS based on the available capacity in each Availability Zone. The [[RDS_Instance_Types|internals]] involve spare [[ec2]] computing resources that are made available for users to purchase at a lower cost. These instances can be interrupted when there is high demand or insufficient capacity.

When designing global-scale systems using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]], you must consider their regional nature. This means that [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] are only available within a specific AZ, which could lead to unbalanced resource distribution if not carefully designed. To overcome this limitation, you can implement instance fleets that span multiple AZs, ensuring higher availability. Instance fleets allow you to request different combinations of instances, enabling better workload distribution, and providing fallback options during interruptions.

### Comparison & Anti-Patterns

| Service          | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]                         | Alternatives                              |
| --------------- | --------------------------------------- | ------------------------------------------ |
| Ideal Use Case   | Short-term, fault-tolerant, flexible    | Long-term, predictable, mission-critical   |
| Anti-pattern    | Mission-critical workloads             | Reserved/On-Demand instances               |
| Design Mistakes | Overlooking interruption notices        | Not implementing throttling mechanisms      |
|                 | Inadequate monitoring                   | Ignoring quota [[Git_hub_notes/certified-aws-solutions-architect-professional-main/12-security-and-config/cloudhsm|limitations]]                   |

### [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created to manage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] access. Here's an example policy JSON snippet allowing [[ec2]] actions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]:
```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:RequestSpotInstances",
    "ec2:ModifySpotInstanceAttributes",
    "ec2:TerminateInstances"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "ec2:instance-action": [
        "request",
        "hibernate",
        "resume-from-checkpoint",
        "modify-attributes",
        "terminate"
      ]
    }
  }
}
```
Cross-account access can be established via resource-based [[policies]] like this one:
```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
      "Action": "ec2:DescribeSpotInstanceRequests",
      "Resource": "arn:aws:ec2:us-west-2:111122223333:spot-instancerequest/*"
    }
  ]
}
```
Organization Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] usage across accounts. For instance, setting up an [[SCP]] to deny specific API calls related to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]].

### Performance & Reliability
Throttling limits are essential to avoid hitting API rate limits when working with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]. Implementing exponential backoff strategies helps mitigate [[api-gateway|errors]] due to throttling.

HA/DR patterns include using instance fleets across multiple AZs, maintaining standby instances ready to take over upon failure, and leveraging services such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] DNS failover and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ deployments.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be applied through tagging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] and creating custom [[billing]] Reports. Calculating potential savings involves comparing the Spot Price to the On-Demand price, considering the possible interruption rates.

For example, let's say you have a c5.large instance type running for 24 hours in the us-east-1 region. Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] would result in approximately $0.034 per hour compared to $0.133 per hour for On-Demand instances. However, these costs may vary depending on the region and time.

### Professional Exam Scenario 1
Question: Which architecture strategy provides the best solution for a genome processing application that requires large-scale compute resources but has unpredictable demand?

Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] with instance fleets spanning multiple AZs and Auto Scaling Groups (ASGs) would provide the most cost-effective solution while maintaining high availability. ASGs will handle scaling up and down the number of instances based on demand.

### Professional Exam Scenario 2
Question: How do you ensure secure cross-account access to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] when using a centralized master account managing several restricted accounts?

Answer: Create a role in the master account with permissions to perform required [[ec2]] actions on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]], then create a resource-based policy granting the necessary permissions to the restricted accounts' [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities. Additionally, apply Service Control [[policies]] (SCPs) at the organization level to restrict any unwanted changes in the restricted accounts.