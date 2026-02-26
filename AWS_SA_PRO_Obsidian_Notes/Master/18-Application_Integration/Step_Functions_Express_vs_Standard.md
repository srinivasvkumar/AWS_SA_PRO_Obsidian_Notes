## Advanced Architecture

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] is a serverless orchestration service that allows developers to coordinate multiple AWS services into serverless applications. It provides two types of runtimes: Express and Standard.

Express workflows have these characteristics:
- Timeout is 1 minute.
- Supports up to 256 MB of input/output data.
- Uses JSON-based input/output data.
- Execution is terminated if there is an error in any task.

Standard workflows have these characteristics:
- Configurable timeout between 1 second and 1 year.
- Supports up to 31,104 KB (31.7 MB) of input/data and 62,208 KB (63.5 MB) of output data.
- Allows for more advanced features like error handling, retry, catch, and timeouts at the state level.
- Supports JSON, string, binary, and object-map data types as input/output data.

[[RDS_Instance_Types|Global scale considerations]] include:
- Express Workflows can be used across regions by configuring [[dynamodb|DynamoDB Global Tables]] for state storage.
- Standard Workflows support cross-region replication using Amazon [[eventbridge]].

## Comparison & Anti-Patterns

| Service | Use Case |
|---|---|
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] Express | Short duration tasks (< 1 min), low data payload (< 256MB) |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] Standard | Longer running processes (> 1 min), large data payload (up to 31.7MB), complex business logic |
| [[lambda|AWS Lambda]] | Direct API calls, small data payload (< 6MB), short duration tasks |
| [[aws-batch|AWS Batch]] | Data processing, long running tasks, high memory requirements |

Anti-patterns include:
- Using Express when Standard would provide more functionality.
- Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] for real-time streaming data processing.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created using JSON policy statements. Here's an example granting access to start an execution in a specific role:
```json
{
    "Effect": "Allow",
    "Action": [
        "states:StartExecution"
    ],
    "Resource": "arn:aws:states:*:123456789012:stateMachine:HelloWorld",
    "Condition": {
        "StringEquals": {
            "states:iamRole": "arn:aws:iam::123456789012:role/service-role/MyStateMachineServiceRole"
        }
    }
}
```
Cross-account access involves creating resources in both accounts, setting appropriate permissions, and then referencing the ARN from the other account.

Organization Service Control [[policies]] (SCPs) can enforce restrictions on usage of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]. For instance, you could deny all `states:CreateStateMachine` requests except those made by certain [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

## Performance & Reliability

Throttling limits depend on the type of workflow:
- Express: 10,000 executions per account per region, 1,000 executions per second.
- Standard: 10,000 executions per account per region, 1,000 executions per second.

Exponential backoff strategies should be implemented when dealing with [[api-gateway|errors]]. This involves waiting twice as long before each subsequent request until reaching a maximum wait time.

HA/DR patterns typically involve using multiple active workflows in different regions, which increases redundancy but also cost.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved through tagging. Tagging enables you to track costs associated with specific projects, teams, or applications. Remember, though, that tags applied to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] don't propagate to underlying resources.

Cost calculation examples:
- Express: $0.00000025 per 1,000 requests + $0.00000004 per state transition.
- Standard: $0.00000040 per 1,000 requests + $0.00000004 per state transition.

## Professional Exam Scenario

### Scenario 1

Question: Which solution meets the following requirements?
1. Orchestrate five different AWS services.
2. Handle data payloads larger than 256MB.
3. Ensure fault tolerance by distributing workloads across regions.

Answer: Solution A - Standard Workflows with [[dynamodb]] Global Table for state storage.

Justification:
- Standard Workflows support complex business logic and large data payloads.
- Distributing workloads across regions requires using Global Tables for state storage.

Incorrect Answers:
- Solution B: Express Workflows cannot handle data payloads larger than 256MB.
- Solution C: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] do not distribute workloads across regions unless specifically designed to do so.

### Scenario 2

Question: What measures should be taken to minimize costs while maintaining performance and reliability for a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] application that runs every hour and has a 30-minute timeout?

Answer:
1. Schedule the workflow to run every hour using [[cloudwatch]] Events instead of continuously polling.
2. Implement exponential backoff strategies for error handling.
3. Utilize tagging for better cost tracking and reporting.

Justification:
- Scheduling reduces unnecessary executions.
- Exponential backoff minimizes throttling and potential downtime due to [[api-gateway|errors]].
- Tagging enables granular cost control and optimization.