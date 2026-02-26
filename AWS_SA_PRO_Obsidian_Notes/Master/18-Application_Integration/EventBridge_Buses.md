**Advanced Architecture: [[eventbridge]] Buses**

[[eventbridge]] operates at two levels: events and rules. Events are categorized into event buses, which can be standard or custom. Standard event buses have predefined events from built-in services like [[ec2]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and [[lambda]]. Custom event buses enable you to receive events from custom applications and services.

Architecturally, an [[eventbridge]] event bus is a logical construct that receives events from various sources. It has a unique name within its account and region, and it can be shared across accounts as a partner event bus. This sharing enables cross-account publishing and subscribing, allowing centralized management of common event types.

Internally, [[eventbridge]] uses a combination of Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Simple Notification Service (SNS)]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Simple Queue Service (SQS)]] under the hood to provide reliable event delivery. The [[eventbridge]] infrastructure ensures global scale and high availability by replicating the underlying resources in multiple AZs.

**Comparison & Anti-Patterns**

| Service          | Comparison                                                                 |
| --------------- | -------------------------------------------------------------------- |
| [[step-functions|AWS Step Functions]] | Best used when designing stateful workflows, not for simple event-driven scenarios. |
| Amazon [[sqs]]       | Primarily designed for message queues, not for event routing.               |
| Amazon [[sns]]       | Provides publish/subscribe capabilities but lacks advanced filtering options. |

Anti-pattern: Using [[eventbridge]] for long-running workflows without using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]].

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Effect": "Allow",
    "Action": [
        "events:PutEvents"
    ],
    "Resource": "arn:aws:events:*:*:event-bus/my-custom-bus"
}
```
Cross-Account Access:
```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": [
        "events:PutEvents"
    ],
    "Resource": "arn:aws:events:*:*:event-bus/my-partner-bus",
    "Condition": {
        "StringEquals": {
            "aws:SourceVpc": "vpc-12345678"
        }
    }
}
```
Organization SCPs:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyPublishToUnapprovedBus",
            "Effect": "Deny",
            "Action": [
                "events:PutEvents"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIgnoreCase": {
                    "aws:SourceVpc": [
                        "vpc-approved-1",
                        "vpc-approved-2"
                    ]
                }
            }
        }
    ]
}
```
**Performance & Reliability**

[[eventbridge]] provides throttling limits based on the number of events per second for each event bus. If your application exceeds these limits, implement exponential backoff strategies to handle throttled requests.

HA/DR patterns:

* Enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] to create recovery points for [[eventbridge]] resources.
* Implement Multi-Region replication for custom event buses and share them with other regions.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring event usage through [[cloudwatch]] metrics and setting up [[billing]] alarms. Calculate costs using the following formula:

`(Number of events * Cost per event) + (Number of rules * Cost per rule)`

**Professional Exam Scenarios**

Scenario 1: Your organization needs to manage events across multiple accounts. However, they want to restrict specific users from publishing events to certain event buses. How would you achieve this?

Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that allow PutEvents actions only for specific event buses and attach those [[policies]] to relevant user roles. Additionally, enforce these restrictions using [[organizations]] SCPs to deny PutEvents actions for unapproved event buses.

Correct answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and [[organizations]] SCPs.
Incorrect answer: Create a single event bus with all required permissions and grant access to specific users.

Scenario 2: Your system publishes millions of events daily, causing occasional throttling issues. How would you address this problem while maintaining reliability and minimizing costs?

Implement exponential backoff strategies to handle throttled requests and optimize the number of rules and events to minimize costs. Consider alternative solutions if the issue persists, such as using Amazon [[sns]] for pub/sub messaging or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] for stateful workflows.

Correct answer: Implement exponential backoff strategies and optimize rules and events.
Incorrect answer: Increase throttling limits and add more event buses.