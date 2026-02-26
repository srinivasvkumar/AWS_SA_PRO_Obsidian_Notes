**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[eventbridge]] (formerly [[cloudwatch]] Events) is a serverless event bus service that makes it easy to connect applications together using data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services. It enables decoupling of event sources and targets, so they don't need to know about each other. The architecture consists of three main components: event buses, rules, and targets.

Internally, [[eventbridge]] uses a centralized event routing system that operates at global scale. This allows events to be propagated across regions when needed, without requiring manual intervention. Under the hood, [[eventbridge]] maintains an index of all events, which enables efficient querying and filtering based on specific criteria.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service                      | Use Case                                                              |
| ---------------------------- | -------------------------------------------------------------------- |
| [[eventbridge]]                | Orchestration, workflow automation, real-time file processing        |
| [[sns]] Topics                   | Asynchronous message processing, fan-out communication             |
| [[sqs]] Queues                   | Backpressure-tolerant queuing, guaranteed delivery                 |
| [[lambda]] Destinations          | Serverless invocation, fine-grained control over retries            |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]               | Stateful orchestration, error handling, long-running processes       |

Anti-patterns include:

- Overuse of [[eventbridge]] as a messaging system instead of using more appropriate services like [[sns]] or [[sqs]].
- Tightly coupling source and target systems within [[eventbridge]] rules.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] allow granular access management in [[eventbridge]]. For cross-account access, you can create an event bus in one account and grant permission to another account to put events onto it. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": [
                "events:PutEvents"
            ],
            "Resource": "arn:aws:events:*:111122223333:event-bus/my-event-bus"
        }
    ]
}
```

For further [[appsync|security]], Organizational Service Control [[policies]] (SCPs) can be used to restrict creation of [[eventbridge]] resources only within specified OUs or accounts.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[eventbridge]] has throttling limits per rule, set at 600 events per minute by default. To handle high traffic scenarios, implement exponential backoff strategies using client libraries that automatically retry requests when necessary.

HA/DR patterns involve configuring [[eventbridge]] in multiple regions, allowing automatic failover and ensuring continuous operation during outages.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved by enabling Data Alarms in [[eventbridge]], which sends notifications whenever usage exceeds certain thresholds. This helps identify potential areas for optimization and cost reduction.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

*Scenario A*: An e-commerce company wants to process incoming orders from their application and send them to various downstream systems for order fulfillment, inventory management, and customer notifications. Design a solution using [[eventbridge]] while avoiding common anti-patterns.

Correct answer: Create an event bus named `order-events`. Set up rules to match specific event patterns related to new orders. Configure targets such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] for order processing, [[sns]] topics for fan-out communication, and [[kinesis|Kinesis Data Streams]] for real-time file processing. Ensure loose coupling between sources and targets.

Incorrect answer: Directly invoke downstream systems from the event rules without proper abstraction layers or decoupling mechanisms.

*Scenario B*: A media organization needs to manage costs associated with [[eventbridge]] usage across multiple accounts within their [[AWS Organization]]. Implement a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] strategy.

Correct answer: Enable Data Alarms in [[eventbridge]] to monitor usage and receive notifications when usage exceeds predefined thresholds. Utilize [[billing|AWS Budgets]] to set custom [[Budgets]] and alerts for [[eventbridge]] spending. Apply Service Control [[policies]] (SCPs) to limit [[eventbridge]] resource creation to specific OUs or accounts.

Incorrect answer: Ignore monitoring and controlling [[eventbridge]] usage without implementing any [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] measures.