**Advanced Architecture: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] Activities [[RDS_Instance_Types|Internals]]**

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] is an orchestrator of AWS services that enables building distributed systems using visual workflows. An activity represents a unit of work in a state machine that can be performed by another service, such as an [[lambda|AWS Lambda]] function, an Amazon [[ecs]] task, or even a manual step. Activities offer durability, scalability, and high availability through the use of Amazon Simple Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]) for storing execution data and Amazon [[dynamodb]] for maintaining state.

[[RDS_Instance_Types|Global scale considerations]] include regional [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], cross-region replication, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]. As of now, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] is available in multiple regions, but not all services integrated with activities are supported globally. For cross-region replication, you must build your application architecture accordingly, leveraging features like [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 replication]] and [[dynamodb|DynamoDB global tables]]. In case of a region-wide outage, you can set up backup executions in another region using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or AWS Server Migration Service.

Under the hood, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] uses Activity [[step-functions|States]] to manage activities. The service polls the target service using long-polling until it receives a result or times out. Once the response is received, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] stores the output in the associated execution history. If the activity does not complete within the timeout period, the execution enters a failed state.

**Comparison & Anti-Patterns: When NOT to use this service**

| Reason                            | Alternatives                                                             |
|-----------------------------------|--------------------------------------------------------------------------|
| Real-time processing              | [[appsync|AWS AppSync]], [[iot|AWS IoT]] Rules Engine, AWS [[eventbridge]]                |
| Custom scheduling                | AWS [[cloudwatch]] Events, AWS [[eventbridge]], [[aws-batch|AWS Batch]]                      |
| Stateful file processing          | [[glue|AWS Glue]], [[Git_hub_notes/certified-aws-solutions-architect-professional-main/08-data-analytics/others|AWS Data Pipeline]]                                           |
| Direct container orchestration   | Amazon [[ecs]], Amazon [[eks]], AWS [[Fargate]]                                    |
| Low-level control over resources   | AWS SDK, AWS CLI, [[AWS CDK]], [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]]                         |

Common anti-patterns include:

* Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] solely for fan-out/fan-in scenarios instead of AWS [[api-gateway|API Gateway]] or AWS Amplify.
* Implementing complex business logic in activities rather than higher-level services like [[lambda|AWS Lambda]] or [[appsync|AWS AppSync]].
* Overusing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] when simple queueing mechanisms like Amazon [[sqs]] or direct API calls would suffice.

**[[appsync|Security]] & Governance: Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], Cross-Account Access, and Organization SCPs**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow the principle of least privilege. Here's an example policy allowing a user to start and list [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] executions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["states:StartExecution", "states:ListExecutions"],
      "Resource": ["arn:aws:states:*::stateMachine:*"]
    }
  ]
}
```

Cross-account access involves creating resource-based [[policies]] on the source account and assuming roles from the destination account. For example, here's a resource-based policy granting `arn:aws:iam::<destination_account>:root` access to a state machine:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "<source_account_ID>"},
      "Action": ["states:StartExecution"],
      "Resource": "arn:aws:states:<region>::stateMachine:<stateMachineName>",
      "Condition": {"StringEquals": {"states:clientToken": "$$.x-amz-client-token"}}
    }
  ]
}
```

Organization Service Control [[policies]] (SCPs) can enforce restrictions across accounts. For instance, you could create an [[SCP]] denying specific actions like `states:UpdateStateMachine`.

**Performance & Reliability: Throttling Limits, Exponential Backoff Strategies, and HA/DR Patterns**

Throttling limits depend on the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] pricing tier. To mitigate throttling issues, implement exponential backoff strategies with jitter to ensure efficient handling of failures.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute applications across multiple regions and use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and failover [[policies]]. Additionally, maintain standby executions in other regions using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or AWS Server Migration Service.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]: Granular Cost Controls and Calculation Examples**

Granular cost controls involve setting [[billing]] alerts based on usage, enabling [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] provisioned concurrency for faster state machine starts, and selecting suitable pricing tiers based on requirements.

Calculating costs requires understanding the various components involved, including the number of state transitions, duration of each transition, and the type of state machines used. An example calculation follows:

Suppose you have 100,000 state transitions per month, lasting for 1 minute each, using Express state machines with 20,000,000 requests per month. The estimated monthly cost would be approximately $19.81 ($0.00000125 * 100,000 + $0.0000000025 * 20,000,000).

**Professional Exam Scenario #1**

Scenario: A company has several AWS accounts managing different departments. They want to centralize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] management while enforcing [[appsync|security]] and governance [[policies]]. How would you approach this problem?

Correct answer: Create a centralized [[organizations|AWS Organizations]] master account with a dedicated [[organizations|AWS Organizations]] Service Control Policy ([[SCP]]) restricting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] permissions at the member account level. Set up a shared AWS Single Sign-On (SSO) setup for cross-account access. Utilize AWS Resource Access Manager to share select state machines between accounts.

Incorrect answer: Manage individual AWS Identity and Access Management ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]) [[policies]] in each account without any centralized oversight.

**Professional Exam Scenario #2**

Scenario: A real-time analytics application needs to process data streams using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] before sending them to downstream services. However, Step Funensions' asynchronous nature may introduce delays. What alternative solutions could address this challenge?

Correct answer: Utilize [[appsync|AWS AppSync]] for real-time data processing, or implement custom real-time processing using [[iot|AWS IoT]] Rules Engine or AWS [[eventbridge]].

Incorrect answer: Modify [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]' settings to force synchronous execution, which goes against its intended design and could lead to performance issues.