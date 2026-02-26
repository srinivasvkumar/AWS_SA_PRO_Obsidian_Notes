## Advanced Architecture
At its core, [[billing|AWS Budgets]] is a service that allows users to set custom budgets based on their cost and usage across AWS services. It operates at both account and [[organizations]] levels, supporting multi-account budget strategies. The service internally uses Amazon [[cloudwatch]] to track cost and usage data, which is then aggregated and visualized in the [[billing|AWS Budgets]] console.

Global scale is achieved through regional endpoints and the use of Amazon [[cloudwatch]] data plane operations. Under the hood, [[billing|AWS Budgets]] stores budget data in Amazon [[dynamodb]] for fast and efficient retrieval. This enables the service to handle large volumes of data while maintaining high availability and performance.

## Comparison & Anti-Patterns

| Service          | Use Case                                                              |
|------------------|----------------------------------------------------------------------|
| AWS [[billing|Cost Explorer]]| One-time analysis or exploration of cost and usage data             |
| [[billing|AWS Budgets]]     | Recurring budget monitoring, alerts, and granular cost controls      |
| [[trusted-advisor|AWS Trusted Advisor]]| Identifying underutilized resources and potential cost savings       |

Anti-pattern: Using [[billing|AWS Budgets]] as a standalone tool for one-time cost analysis. Instead, use AWS [[billing|Cost Explorer]] or [[trusted-advisor|AWS Trusted Advisor]] depending on your needs.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created using JSON snippets such as the following example, allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to manage budgets but not modify them:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "budgets:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpce": "vpce-1234567890abcdef0"
                },
                "Bool": {
                    "aws:VpcSourcePermission": "CREATE_SUBNET_ONLY"
                }
            }
        },
        {
            "Effect": "Deny",
            "Action": [
                "budgets:ModifyBudget"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access can be granted by attaching an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to a resource in another account, allowing the principal from the second account to perform specific actions. Additionally, Service Control [[policies]] (SCPs) at the Organization level can enforce budget management [[policies]] across all accounts.

## Performance & Reliability
Throttling limits exist for [[billing|AWS Budgets]] API calls, with a maximum of 20 requests per second. To avoid throttling issues, implement exponential backoff strategies with jitter when making repeated API calls.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] can be ensured by distributing budgets across multiple regions and implementing redundant systems for tracking cost and usage data.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be implemented using [[billing|AWS Budgets]] by setting up budgets for individual services or tags. For example, you could create a budget for [[ec2]] instances tagged with `Environment:production`.

Calculation Example:
Suppose you have an AWS account with the following monthly costs:
- [[ec2]]: $500
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]: $300
- [[lambda]]: $100

You want to set a budget of $800 for this account. Here's how to calculate it:

Total Monthly Cost = [[ec2]] + [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] + [[lambda]]
= $500 + $300 + $100
= $900

Now, create a budget with a limit of $800 and set an alert threshold at 90% utilization. If your costs exceed this limit, [[billing|AWS Budgets]] will trigger an alert.

## Professional Exam Scenarios
### Scenario 1
An organization has multiple AWS accounts managed under a single [[AWS Organization]]. They want to enforce a budget policy across all accounts, ensuring no account exceeds a certain spending limit. How would you implement this?

Correct Answer 1: Implement a Service Control Policy ([[SCP]]) at the Organization level that denies any action that results in budget modifications beyond the specified limit.

Correct Answer 2: Set up a master budget at the Organization level and configure child budgets for each linked account.

Incorrect Answer: Create separate budgets for each account manually without using the Organization features.

### Scenario 2
A company wants to monitor AWS costs for specific teams working on different projects. They have already tagged their resources accordingly. What is the best way to achieve this using [[billing|AWS Budgets]]?

Correct Answer: Create budgets based on these tags, allowing for granular cost monitoring and alerts.

Incorrect Answer: Manually assign resources to different budgets based on project membership.