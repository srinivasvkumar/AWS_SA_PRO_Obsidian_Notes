## Advanced Architecture

At its core, Savings Plans is a flexible pricing model that offers significant discounts compared to On-Demand rates in exchange for a commitment to a consistent amount of compute usage, measured in dollars per hour, for a term of 1 or 3 years. There are two types of Savings Plans: Compute Savings Plans (covering any AWS compute service) and [[ec2]] Instance Savings Plans (covering a specific instance family within a region).

Architecturally, Savings Plans operates at the account level, with an hourly commitment calculated as a 365-day moving average. The service supports multiple accounts through [[organizations|AWS Organizations]], allowing you to manage commitments and [[billing]] centrally. Each plan has its own set of quotas, such as the maximum number of plans, which can be increased by contacting AWS Support.

Internally, Savings Plans uses a sophisticated algorithm to determine the optimal allocation of committed resources across various services and dimensions. This allows for efficient utilization of reserved capacity while maintaining flexibility.

## Comparison & Anti-Patterns

| Service              | Use Case                                                                 |
| -------------------- | ------------------------------------------------------------------------- |
| Savings Plans       | Consistent, predictable workloads spanning multiple services             |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]]  | Single-service, single-family, and single-region workloads                |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]      | Short-term, interruptible, and price-sensitive workloads                   |
| [[Fargate]] Spot        | Stateless, containerized, and short-lived workloads                      |

Anti-patterns include using Savings Plans for highly variable workloads or workloads with unpredictable spikes in demand. In these cases, the cost benefits may not outweigh the risk of underutilizing the reserved capacity.

## [[appsync|Security]] & Governance

To ensure secure cross-account access and centralized management, follow these guidelines:

1. Create a separate master payer account for managing Savings Plans commitments and [[billing]].
2. Enable [[organizations|AWS Organizations]] and create a service control policy ([[SCP]]) to restrict creating new Savings Plans in member accounts.
3. Implement fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON snippets similar to the following example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "savingsplans:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalArn": [
                        "arn:aws:iam::123456789012:role/MySavingsPlansRole"
                    ]
                }
            }
        }
    ]
}
```

4. Leverage AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share Savings Plans across accounts.

## Performance & Reliability

Savings Plans do not have throttling limits or exponential backoff requirements since they operate independently from the underlying services. However, it's essential to monitor usage against committed resources and adjust your Savings Plans accordingly to maintain high performance and reliability.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute Savings Plans across multiple regions based on your organization's needs. Ensure that your application architecture follows [[iam|best practices]] for HA/DR, including redundant infrastructure, data replication, and failover mechanisms.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for Savings Plans involve monitoring and optimizing usage at different levels:

1. Account-level: Track the overall spend and identify underutilized plans.
2. Tagging: Apply tags to Savings Plans and associated resources to track costs by department, team, or project.
3. Alerts: Set up alerts to notify when spending exceeds predefined thresholds.
4. Anomaly detection: Utilize AWS [[billing|Cost Explorer]] anomaly detection to discover unusual usage patterns.

Calculation example:
Suppose you have a 3-year Compute Savings Plan with a commitment of $100,000 annually. To calculate the effective hourly rate, use the following formula:

$100,000 / (365 days × 24 hours) = $115.74 approximately

This means you will receive a discounted hourly rate of approximately $115.74 for any covered AWS compute resource during the 3-year term.

## Professional Exam Scenario

### Scenario 1

Your organization runs several machine learning workloads on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]]. After analyzing the monthly bills, you notice that the ML workloads contribute significantly to the total AWS spend. Identify the most suitable Savings Plans option(s) for reducing the ML workload costs and explain your reasoning.

Correct answer(s):

1. Compute Savings Plans: Since ML workloads span multiple services, including [[ec2]] instances and [[Fargate]], Compute Savings Plans provide flexibility and cost savings.
2. [[ec2]] Instance Savings Plans (optional): If the ML workloads primarily utilize specific instance families, additional cost savings might be achieved through the use of [[ec2]] Instance Savings Plans.

Incorrect answer(s):

1. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]]: Not suitable because ML workloads typically require multiple instance families and sizes, making them less compatible with RIs.
2. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]: While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] can reduce costs, they are not ideal for production ML workloads due to their interruption-prone nature.

### Scenario 2

You want to implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] for a multi-account environment with a centralized [[billing]] system. Describe how you would leverage Savings Plans, [[organizations|AWS Organizations]], and AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to achieve this goal.

Implementation steps:

1. Create a master payer account and enable [[organizations|AWS Organizations]].
2. Create a service control policy ([[SCP]]) to restrict creating new Savings Plans in member accounts.
3. Implement fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] in the master payer account, allowing relevant roles to interact with Savings Plans.
4. Designate one or more master accounts responsible for managing Savings Plans commitments and [[billing]].
5. Share Savings Plans with member accounts using AWS [[ram]].

By implementing this solution, you can effectively manage Savings Plans at scale while ensuring centralized cost tracking and optimization.