**Advanced [[eventbridge]] (Serverless Bus) Architecture**

[[RDS_Instance_Types|Internals]]: [[eventbridge]] is a serverless event bus that enables decoupling of applications and services to build a highly responsive microservices architecture. It has two types of event buses: default and custom. The default event bus receives events from enabled Amazon sources such as [[ec2]], [[sns]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Custom event buses receive events from custom applications and services. [[eventbridge]] supports rule-based filtering patterns and schedules to enable or disable rules.

[[RDS_Instance_Types|Global Scale Considerations]]: [[eventbridge]] is available in multiple regions and supports cross-region events using partner event buses. This allows for building globally distributed systems that can handle large-scale workloads. However, it's important to consider the latency introduced by cross-region invocations.

Under the Hood Mechanics: [[eventbridge]] uses an underlying message broker to manage the delivery of events to subscribers. It ensures at-least-once delivery semantics, meaning that some events may be delivered more than once. To avoid duplicates, implement idempotent receivers.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[eventbridge]] | Decoupled microservices, real-time file processing, scheduled tasks |
| [[sqs]] | FIFO order, guaranteed delivery, strict ordering requirements |
| [[kinesis|Kinesis Data Streams]] | Real-time data collection, streaming data processing |
| [[sns]] | Pub/Sub messaging, broadcasting messages to multiple subscribers |

Anti-pattern: Using [[eventbridge]] for synchronous request/response scenarios, as it doesn't guarantee message ordering. Instead, consider using [[api-gateway|API Gateway]], [[lambda]], or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]].

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "events:PutEvents"
            ],
            "Resource": [
                "arn:aws:events:*:123456789012:event-bus/my-custom-bus"
            ]
        }
    ]
}
```
Cross-Account Access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "123456789012"
            },
            "Action": [
                "events:PutEvents"
            ],
            "Resource": "arn:aws:events:us-west-2:111122223333:event-bus/default",
            "Condition": {
                "ArnLike": {
                    "aws:SourceAccount": "123456789012"
                }
            }
        }
    ]
}
```
Organization SCPs:
```json
{
    "Sid": "DenyUnapprovedEventBusCreation",
    "Effect": "Deny",
    "Scope": {
        "FilteredResourceIds": [
            "event-bus/*"
        ]
    },
    "Principal": "*",
    "Action": "events:CreateEventBus",
    "Condition": {
        "StringNotEquals": {
            "aws:PrincipalOrgID": "o-123456789012"
        }
    }
}
```
**Performance & Reliability**

Throttling Limits: [[eventbridge]] has a default limit of 10,000 events per second. If you need higher throughput, contact AWS support to increase your limits.

Exponential Backoff Strategies: Implement exponential backoff when handling throttled events. For example, start with a base delay of 1 second and double the delay with each retrial up to a maximum delay of 60 seconds.

HA/DR Patterns: Use [[cloudwatch]] Events to schedule backups, maintenance, and other routine tasks across multiple regions.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls: Enable [[eventbridge]] metrics and alarms to monitor usage costs. Set up [[billing]] alerts to notify you if your monthly spending exceeds a certain threshold.

Calculation Examples:

* $1 per month (Basic [[eventbridge]] plan)
* $0.0011 per event (Additional events)
* $0.0000001 per event (Retained custom event pattern)

**Professional Exam Scenarios**

Scenario 1: A company wants to process images uploaded to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] in real time. They want to trigger a [[lambda]] function whenever a new image is uploaded.

Correct Answer: Configure an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket notification to send an event to [[eventbridge]] and create a rule to invoke the [[lambda]] function.

Incorrect Answer: Directly configure the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket to trigger the [[lambda]] function. This approach doesn't allow for additional filters, and it prevents the event from being propagated to other services.

Scenario 2: An organization wants to enforce [[eventbridge]] usage restrictions within their [[organizations|AWS Organizations]] account.

Correct Answer: Create an Organization Service Control Policy ([[SCP]]) to deny specific [[eventbridge]] actions and resources.

Incorrect Answer: Rely solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. This approach does not provide centralized control over [[eventbridge]] usage across all accounts in the organization.