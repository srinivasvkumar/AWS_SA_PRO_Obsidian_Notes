Advanced Architecture
---------------------

At its core, Health Dashboard is a regional service that provides visibility into the health of various AWS services within an account or across an entire organization. It aggregates data from [[cloudwatch]], Trusted Advisor, Service Health Dashboard, and Personal Health Dashboard to offer a comprehensive view of the health of your AWS resources.

Health Dashboard operates at both the account and organizational level. The service can monitor the health of resources in multiple accounts by using [[organizations|AWS Organizations]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Health Dashboard uses Amazon [[cloudwatch]] Events to collect data about resource health and alarms. This data is then processed and displayed in the Health Dashboard interface.

To support global scale, Health Dashboard employs a highly available and scalable architecture based on Amazon [[dynamodb]], Amazon [[sns]], and [[lambda]]. When a resource's health changes, [[cloudwatch]] publishes an event to an [[sns]] topic, which triggers a [[lambda]] function. The [[lambda]] function updates the resource's status in [[dynamodb]], ensuring that the information in Health Dashboard remains up-to-date.

Comparison & Anti-Patterns
---------------------------

| Service | Use cases |
| --- | --- |
| Health Dashboard | Monitoring the health of AWS resources and services across accounts in an [[AWS Organization]] |
| [[cloudwatch|CloudWatch Alarms]] | Creating custom alarms and notifications based on specific [[cloudformation|conditions]] |
| Service Health Dashboard | Viewing the availability and performance of individual AWS services |
| Personal Health Dashboard | Receiving personalized notifications about events affecting your AWS resources |

Anti-pattern: Using Health Dashboard as a replacement for [[cloudwatch|CloudWatch Alarms]] or vice versa. These services have different purposes and should be used together for optimal monitoring and alerting.

[[appsync|Security]] & Governance
----------------------

Health Dashboard supports cross-account access through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[organizations|AWS Organizations]]. To enable Health Dashboard to access resources in another account, perform the following steps:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to access Health Dashboard.
2. Attach the necessary managed [[policies]] (e.g., `AWSHD.IntegrationRole`).
3. Add a trust relationship allowing the Health Dashboard service principal (`healthdashboard.amazonaws.com`) to assume the role.
4. In the target account, grant the necessary permissions to the Health Dashboard [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.

You can also enforce usage of Health Dashboard via Service Control [[policies]] (SCPs) in [[organizations|AWS Organizations]]. For example, you could create an [[SCP]] that denies actions related to other monitoring services like [[cloudwatch|CloudWatch Alarms]].

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "cloudwatch:*"
            ],
            "Resource": "*"
        }
    ]
}
```

Performance & Reliability
--------------------------

Health Dashboard has built-in throttling limits to prevent overloading the system. If you exceed these limits, you may receive a `ThrottlingException`. To avoid this issue, implement exponential backoff strategies when interacting with Health Dashboard.

HA/DR patterns involve deploying Health Dashboard in multiple regions and replicating data between them. However, since Health Dashboard is a regional service without native multi-region capabilities, implementing HA/DR requires additional infrastructure such as custom scripts and third-party tools.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
-----------------

There are no direct costs associated with Health Dashboard. However, enabling Health Dashboard increases the usage of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] (e.g., [[cloudwatch]], [[sns]], [[lambda]], and [[dynamodb]]). Therefore, it's important to monitor the overall costs associated with these services and optimize their usage accordingly.

Professional Exam Scenarios
---------------------------

Scenario 1: A company wants to centralize monitoring of all its AWS accounts using Health Dashboard. They have 50 accounts distributed across three AWS Regions. How would you implement this solution?

Correct answer: Implement [[organizations|AWS Organizations]] and attach the Health Dashboard [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in each account. Grant Health Dashboard the required permissions to access resources in the other accounts. This way, Health Dashboard will consolidate monitoring data for all 50 accounts.

Incorrect answer: Implement a single Health Dashboard instance in one region and allow access from the other two regions.

Justification: Allowing access from other regions does not provide centralized monitoring. Each region would still require manual configuration and maintenance.

Scenario 2: An enterprise wants to ensure that only Health Dashboard is used for monitoring purposes. How would you achieve this using Service Control [[policies]]?

Correct answer: Create an [[SCP]] that denies actions for other monitoring services like [[cloudwatch|CloudWatch Alarms]].

Incorrect answer: Implement a custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that restricts users from creating [[cloudwatch|CloudWatch Alarms]].

Justification: Custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] cannot be applied across an entire organization. SCPs are more suitable for enforcing restrictions at scale.