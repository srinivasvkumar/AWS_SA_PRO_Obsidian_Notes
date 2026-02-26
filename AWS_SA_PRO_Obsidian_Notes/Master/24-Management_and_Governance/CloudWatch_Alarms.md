**Advanced Architecture: [[cloudwatch|CloudWatch Alarms]] [[RDS_Instance_Types|Internals]], [[RDS_Instance_Types|Global Scale Considerations]], and 'Under the Hood' Mechanics**

[[cloudwatch|CloudWatch Alarms]] monitor metrics about resources and applications over a specified period of time, and perform one or more actions based on the value of a metric relative to a threshold you define. The alarms send notifications via [[sns]] topics or Auto [[asg|Scaling policies]]. They can also be used to stop, terminate, reboot, or recover instances.

Internally, [[cloudwatch]] Alarm evaluates the alarm state based on the metric data returned by PutMetricData API calls or other sources like logs, collectd, and [[cloudwatch|CloudWatch Logs]]. It calculates the breaching threshold using statistics (minimum, maximum, average) and performs periodic evaluations.

[[cloudwatch|CloudWatch Alarms]] support cross-region and cross-account aggregation allowing users to create alarms across multiple regions and accounts. This enables central monitoring of distributed systems at scale. However, it is essential to understand that [[cloudwatch|CloudWatch Alarms]] do not currently support inter-region metrics transfer. Metrics must be available in each region where an alarm is needed.

**Comparison & Anti-Patterns: When NOT to use [[cloudwatch|CloudWatch Alarms]] and Common Design Mistakes**

| Service                    | Use Case                                                                   |
| ---------------------------| ------------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]                | Operational/Audit logs                                                     |
| [[config]] Rules              | Configuration drift detection                                               |
| [[eventbridge]] Rules          | Time-based triggering                                                      |
| [[sqs]] / [[sns]] / [[kinesis|Kinesis Data Firehose]] | Streaming data processing                                             |

Common anti-patterns include setting alarms too broadly (e.g., all AWS services in a single alarm), using insufficient granularity (<5m periods), and failing to account for daily/weekly fluctuations in traffic patterns.

**[[appsync|Security]] & Governance: Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] (JSON Snippets), Cross-Account Access, and Organization SCPs**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] control who can manage which alarms. Here's an example JSON policy granting permissions to describe and modify alarms within a specific namespace:

```json
{
    "Effect": "Allow",
    "Action": [
        "cloudwatch:DescribeAlarms",
        "cloudwatch:PutMetricAlarm"
    ],
    "Resource": [
        "*:*/*",
        "arn:aws:cloudwatch::account-id:alarm:*MyAppMetrics*"
    ]
}
```

Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles setup. For instance, a user from Account A wanting to create an alarm on a metric in Account B needs to have `cloudwatch:PutMetricData` permission in Account B and `cloudwatch:PutEvaluatedAlarmState` permission in Account A.

Organization Service Control [[policies]] (SCPs) provide organization-wide management of permissions. To enforce centralized control over [[cloudwatch|CloudWatch Alarms]], [[organizations]] can set up SCPs restricting certain APIs or limiting their usage to specific IP ranges.

**Performance & Reliability: Throttling Limits, Exponential Backoff Strategies, and HA/DR Patterns**

[[cloudwatch|CloudWatch Alarms]] have throttling limits such as 20 requests per second for PutMetricData. High request rates may result in [[api-gateway|errors]] due to throttling. Implementing exponential backoff strategies helps handle these situations.

HA/DR patterns typically involve creating [[cloudwatch|CloudWatch Alarms]] in both primary and secondary regions. In case of failover, alarms should switch targets accordingly.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]: Granular Cost Controls and Calculation Examples**

Granular cost controls can be achieved by setting alarms based on specific metrics rather than generic ones. For example, instead of setting an alarm on total CPU utilization, set it on individual [[ec2]] instance CPU [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] balance.

Calculating costs involves understanding billable units. [[cloudwatch]] charges $0.10 per month for each custom metric. If you publish 100 custom metrics every minute throughout the month, your monthly cost would be approximately $175.

**Professional Exam Scenario 1**

You are designing a system for a client that uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] extensively. The client wants to receive alerts when any function exceeds its memory limit consistently. What solution would you recommend?

Correct Answer: Create a [[cloudwatch]] Alarm that triggers whenever the "Maximum memory used (MB)" metric for the [[lambda]] function goes above a defined threshold.

Incorrect Answers:
- Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] doesn't track [[lambda]] metrics.
- Creating a rule in [[eventbridge]]: While useful for scheduling tasks, it doesn't help monitor [[lambda]] metrics directly.

**Professional Exam Scenario 2**

Suppose you work for a managed service provider managing multiple clients' AWS accounts. You want to ensure no unauthorized changes occur to [[cloudwatch|CloudWatch Alarms]] across all these accounts. How would you implement this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] and [[organizations]]?

Correct Answer: Set up an Organization [[SCP]] denying PutEvaluatedAlarmState action except for specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities. Additionally, configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles allowing necessary PutMetricData actions only for relevant resources.

Incorrect Answers:
- Setting up a Service Control Policy ([[SCP]]) denying PutMetricData: This prevents sending data but allows modifying existing alarms.
- Adding a deny-all policy at the root level: This blocks everything, including desired actions.