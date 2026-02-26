**Advanced Architecture: [[eventbridge]] Pipes [[RDS_Instance_Types|Internals]], Global Scale Considerments, and 'Under the Hood' Mechanics**

Architecture:
- [[eventbridge]] Pipes is an extension of [[eventbridge]] that enables the creation of serverless pipelines between various services.
- It uses two primary components: input transformers and output targets.
- Input transformers convert incoming events into a format suitable for the output target.
- Output targets receive and process transformed events.

[[RDS_Instance_Types|Internals]]:
- Events flow through EventBrule Pipes using JSON Path expressions and template-based transformations.
- For [[RDS_Instance_Types|global scale considerations]], you can create rules in multiple regions and replicate them across accounts.
- [[eventbridge]] Pipes supports cross-region and cross-account event routing.

[[RDS_Instance_Types|Global Scale Considerations]]:
- To enable global scale, set up centralized [[eventbridge]] buses in a single region or multiple regions.
- Implement regional [[eventbridge]] buses for localized processing and failover scenarios.
- Leverage [[eventbridge]] Pipes' cross-region and cross-account capabilities for multi-region data distribution and processing.

**Comparison & Anti-Patterns: When NOT to use this service vs. alternatives (Table format)**

| Service                      | When to choose                |
|------------------------------|------------------------------|
| [[eventbridge]] Pipes            | Multi-service integration   |
| [[sqs]]                          | Queue management             |
| [[lambda]]                       | Custom logic execution        |
| [[kinesis|Kinesis Data Streams]]         | Real-time streaming           |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]              | Serverless workflows         |
| [[sns]]                          | Pub/Sub messaging             |

Anti-Patterns:
- Using [[eventbridge]] Pipes for non-event-driven use cases.
- Replicating existing functionality from other services without understanding benefits.

**[[appsync|Security]] & Governance: Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] (JSON Snippets), Cross-Account Access, and Organization SCPs**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy Example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "events:PutEvents"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-Account Access:
- Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions to allow cross-account access.
- Grant necessary actions to specific users, groups, or roles.

Organization SCPs:
- Enforce [[control-tower|guardrails]] by setting Service Control [[policies]] at the organization level.
- Prevent unwanted changes or actions within [[eventbridge]] Pipes.

**Performance & Reliability: Throttling Limits, Exponential Backoff Strategies, and HA/DR Patterns**

Throttling Limits:
- Understand throttling limits and implement appropriate retry mechanisms.
- Refer to official documentation for current limits.

Exponential Backoff Strategies:
- Implement exponential backoff strategies when handling throttled requests.
- Increase delay intervals after each failed request to avoid overloading the system.

HA/DR Patterns:
- Set up multi-region [[eventbridge]] buses for high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].
- Utilize [[route53]] [[route53|health checks]] and [[dynamodb]] streams for automatic failover.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]: Granular Cost Controls and Calculation Examples**

Granular Cost Controls:
- Monitor costs using [[cloudwatch|CloudWatch alarms]] and notifications.
- Implement [[billing]] alerts based on custom thresholds.

Calculation Examples:
- Estimate usage and costs based on the number of events processed by [[eventbridge]] Pipes.
- Factor in associated costs from input transformers and output targets.

**Professional Exam Scenario Questions**

Question 1:
Your company operates in multiple regions and needs to distribute events across accounts and regions for a multi-tenant solution. How would you design a solution using [[eventbridge]] Pipes?

Answer:
- Create a central [[eventbridge]] bus in one region and replicate it across other regions as needed.
- Enable cross-region and cross-account event routing for multi-tenant support.
- Use input transformers to convert events into a format suitable for output targets.
- Implement regional [[eventbridge]] buses for localized processing and failover scenarios.

Question 2:
Your organization has strict [[appsync|security]] requirements, and you need to ensure secure communication between services using [[eventbridge]] Pipes. What steps would you take to meet these requirements?

Answer:
- Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions to allow cross-account access.
- Grant only necessary actions to specific users, groups, or roles.
- Set up Service Control [[policies]] (SCPs) at the organization level to enforce [[control-tower|guardrails]].
- Ensure encryption in transit and at rest for sensitive data.