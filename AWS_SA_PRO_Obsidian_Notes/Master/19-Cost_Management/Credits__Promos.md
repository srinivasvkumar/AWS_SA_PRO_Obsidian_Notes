## Advanced Architecture
At the core of AWS [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions is the concept of [[billing]] Conductor, which allows for centralized management and allocation of [[billing]] benefits across multiple accounts. The [[billing]] Conductor operates at the organization level and can manage various types of [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] such as AWS Free Tier, Savings Plans, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs). It also supports promotional [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] like those offered through AWS Activate programs.

[[billing]] Conductor uses [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles to enforce granular control over the distribution and utilization of [[billing]] benefits. For instance, you can create custom SCPs that limit the usage of RIs to specific accounts or regions. Additionally, the [[billing]] Conductor integrates with AWS [[billing|Cost Explorer]] to provide a unified cost management experience.

## Comparison & Anti-Patterns
The following table highlights key differences between AWS [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions and alternative services like [[billing|AWS Budgets]] and [[billing|Cost Explorer]]:

| Feature | AWS [[Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions | [[billing|AWS Budgets]] | [[billing|Cost Explorer]] |
|---|---|---|---|
| **Primary Function** | Centralized [[billing]] benefits management | Setting budgetary alerts and thresholds | Cost and usage analysis and reporting |
| **Target Usage** | Multi-account [[organizations]] | Individual accounts | Individual accounts |
| **Integration with Other Services** | [[billing]] Conductor, [[organizations|AWS Organizations]], [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] | [[billing|Cost Explorer]], Personal Health Dashboard | Trusted Advisor, AWS Support |

Common anti-patterns include:

- Using [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions in place of native [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] techniques like Rightsizing, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]], or Convertible RIs.
- Overcomplicating the structure of SCPs and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles leading to potential misconfiguration and [[appsync|security]] risks.

## [[appsync|Security]] & Governance
To ensure secure access and governance when using [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions, follow these guidelines:

1. Implement strict [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] by leveraging least privilege principles and avoiding overly permissive roles or [[policies]].
2. Create custom SCPs to enforce granular control over the distribution and utilization of [[billing]] benefits. Ensure proper nesting and ordering of SCPs.
3. Monitor and audit the usage of [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] and promotions using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], [[config|AWS Config]], and Amazon [[cloudwatch]] events.

Example JSON policy snippet for cross-account access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
            },
            "Action": [
                "aws-portal:ModifyBillingPermissions",
                "aws-portal:ViewBilling"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpcs": "vpc-XXXXXX"
                }
            }
        }
    ]
}
```

## Performance & Reliability
While there are no specific throttling limits associated with [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions, it is essential to implement exponential backoff strategies when interacting with the [[billing]] API to prevent potential disruptions. This ensures high availability and reliability during periods of high demand or network issues.

HA/DR patterns should involve maintaining backup configurations for the [[billing]] Conductor and ensuring consistent application of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs across primary and secondary environments.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls in [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions include allocating savings plans to specific accounts or groups of accounts within an organization. Calculating the costs associated with [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] and promotions requires understanding the monetary value of each credit type and tracking their consumption over time.

For example, if you have a $1,000 monthly Savings Plan credit and apply it to a single account with a $2,000 monthly usage, the net cost would be calculated as follows:

1. Calculate total charges without applying the Savings Plan credit: $2,000
2. Apply the Savings Plan credit: $2,000 - $1,000 = $1,000

This results in a net cost of $1,000 for the given month.

## Professional Exam Scenarios

### Scenario 1:
Suppose a large enterprise has multiple AWS accounts managed under a single [[AWS Organization]]. They want to allocate RIs and Savings Plans to individual accounts based on their usage patterns while retaining centralized management and monitoring capabilities. Which AWS services should they leverage?

Correct answer: AWS [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|Credits]] & Promotions along with [[billing]] Conductor, [[organizations|AWS Organizations]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]].

Incorrect answer: AWS [[billing|Cost Explorer]] and [[billing|AWS Budgets]] do not support the required functionality.

### Scenario 2:
An organization wants to restrict the use of Savings Plans to only certain AWS Regions while enforcing this restriction centrally. How can they achieve this?

Correct answer: By implementing custom SCPs within [[organizations|AWS Organizations]] and utilizing the [[organizations|AWS Organizations]] integration within the [[billing]] Conductor.

Incorrect answer: Modifying [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or configuring [[billing|AWS Budgets]] will not effectively address this requirement.