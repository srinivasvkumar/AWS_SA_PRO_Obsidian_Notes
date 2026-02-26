### Advanced Architecture
At its core, Savings Plans is a flexible pricing model that offers significant discounts compared to On-Demand prices in exchange for a commitment to a consistent amount of usage, measured in dollars per hour, for a term of 1 or 3 years. There are two main types of Savings Plans: Compute Savings Plans and [[ec2]] Instance Savings Plans. The primary difference between them is that Compute Savings Plans provide flexibility to change instances, operating systems, and regions within the same family, while [[ec2]] Instance Savings Plans offer the highest discounts and apply to a specific instance type within an Availability Zone and region.

Under the hood, Savings Plans aggregates your usage across all accounts within an [[organizations|AWS Organizations]] structure. This allows you to have a unified view of your Savings Plans utilization and ensures that you can efficiently allocate and manage your commitments at scale. Additionally, Savings Plans supports multiple [[billing]] models, including monthly payments, no upfront payments, and all upfront payments. These options enable [[organizations]] to balance their budget requirements and capital expenditure constraints better.

### Comparison & Anti-Patterns
Here's a comparison table between Savings Plans and alternative services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]:

| Service            | Flexibility                        | Upfront Cost   | Duration      |
|--------------------|-----------------------------------|---------------|--------------|
| Savings Plans     | Highly flexible                    | None          | 1 or 3 years  |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]]| Limited flexibility                 | Required      | 1 or 3 years  |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]    | Very limited flexibility           | None          | Variable     |

Common anti-patterns include using Savings Plans when workload patterns are too volatile, making it challenging to meet the required commitment levels. In such cases, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] might be more suitable due to their lower costs and higher flexibility. Another anti-pattern is using Savings Plans without proper monitoring and alerting, which could lead to underutilized plans or unexpected costs.

### [[appsync|Security]] & Governance
To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], you can create a custom policy that grants permissions to purchase and manage Savings Plans. Here's an example JSON snippet demonstrating how to allow users to describe, purchase, and apply Savings Plans:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "savingsplans:Describe*",
                "savingsplans:Purchase*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "savingsplans:Apply*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "savingsplans:ResourceType": [
                        "SavingsPlan::*::*",
                        "SavingsPlan::*:TaggedResource::*"
                    ]
                }
            }
        }
    ]
}
```
For cross-account access, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with a trust relationship policy allowing the target account to perform specific actions. For instance, here's a trust policy allowing the specified target account to describe Savings Plans:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::TARGET_ACCOUNT_ID:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "Bool": {
                    "aws:MSIPAsupported": true
                }
            }
        }
    ]
}
```
You can also enforce Savings Plans adoption using Organization Service Control [[policies]] (SCPs) by restricting specific actions like starting instances without applying a Savings Plan.

### Performance & Reliability
Savings Plans do not have throttling limits or exponential backoff strategies since they don't directly interact with APIs or resources. However, it's essential to monitor Savings Plans utilization and adjust your commitments accordingly to avoid overpaying or underutilizing your plans.

When designing highly available and disaster-resilient architectures, Savings Plans can play a crucial role in reducing costs without compromising performance or availability. By applying Savings Plans to your baseline resources and reserving [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for spiky or non-critical workloads, you can optimize your infrastructure costs significantly.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls are possible through tagging Savings Plans and associating them with specific resources, enabling accurate chargeback and showback reporting. To calculate the cost savings of using Savings Plans, you can compare the effective hourly rate of your Savings Plans against the corresponding On-Demand rate.

For example, if you purchase a Compute Savings Plan with a $0.095/hour rate and the equivalent On-Demand rate is $0.12/hour, you're achieving approximately 20.8% cost savings. Keep in mind that Savings Plans require a minimum commitment level, so ensure you estimate your total commitment accurately based on historical usage data.

### Professional Exam Scenarios
#### Scenario 1:
Your organization has recently started using [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]] to deploy a hybrid cloud environment. You need to minimize the cost of running Outposts while maintaining high availability and reliability. Which strategy should you adopt?

Correct answer: Apply Compute Savings Plans to baseline resources and reserve [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for spiky or non-critical workloads in your Outposts environment.

Incorrect answer: Purchase [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] for your Outposts environment, as RIs lack the necessary flexibility for hybrid environments.

#### Scenario 2:
Your company is expanding its AWS presence into new regions and wants to reduce overall infrastructure costs. You currently rely on [[ec2]] instances and use Savings Plans for your existing workloads. How would you approach [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] for your new regions?

Correct answer: Implement a centralized management system using [[organizations|AWS Organizations]], allowing you to aggregate your Savings Plans usage across all accounts. Monitor Savings Plans utilization and adjust commitments accordingly.

Incorrect answer: Create separate Savings Plans for each new region, increasing administrative overhead and missing out on potential cost savings from global aggregation.