**Advanced Architecture**

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] is a serverless orchestrator that allows developers to coordinate multiple AWS services into serverless workflows. It provides a graphical representation of state machines, which can be written in JSON or YAML and consist of [[step-functions|states]] and their transitions. Understanding the [[RDS_Instance_Types|internals]] of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] requires understanding its components:

* [[step-functions|States]]: The individual elements of a state machine. These include Task, Choice, Parallel, Map, Wait, and Pass.
* Executions: Instances of running state machines.
* History Events: Recorded events during an execution.

Global scale is achieved through regional endpoints and Amazon Resource Names (ARNs) for resources. Each region operates independently, allowing for localized executions. However, there are no global [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]], meaning each region must be managed separately.

Under the hood, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] uses a centralized service called "the Service Coordinator" responsible for managing all executions. The coordinator interacts with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] via APIs and communicates with stateless workers called "Activity Workers" for task completion.

**Comparison & Anti-Patterns**

| Service              | Use Case                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]       | Asynchronous processing, human approvals, choreographing AWS services                    |
| [[lambda|AWS Lambda]]          | Event-driven functions, single-purpose logic                                             |
| Amazon [[dynamodb]]      | Highly available, performant NoSQL database                                               |
| Amazon [[sqs]]           | Durable message queues                                                                     |
| Amazon [[sns]]           | Pub/Sub messaging                                                                         |
| [[glue|AWS Glue]] & [[kinesis]]   | ETL jobs, stream processing                                                              |

Anti-patterns include long-running tasks within [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]. Instead, offload these tasks to services such as [[aws-batch|AWS Batch]] or AWS [[Fargate]]. Also, avoid using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] when performance requirements dictate lower latencies than [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] can provide.

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow the principle of least privilege. For example, allow specific ARNs for Step Function actions:

```json
{
  "Effect": "Allow",
  "Action": [
    "states:CreateActivity*, states:DeleteActivity*, states:DescribeActivity*,",
    "states:ListActivities*"
  ],
  "Resource": [
    "arn:aws:states:*:123456789012:activity:*"
  ]
}
```

Cross-account access is possible through resource-based [[policies]] on the state machine:

```yaml
 policices:
   - Effect: Allow
     Principal:
       AWS: !Sub "arn:aws:iam::${OtherAccountId}:root"
     Action:
       - "states:StartExecution"
     Condition:
       StringEquals:
         "states:targetArn": !Ref MyStateMachine
```

Organization Service Control [[policies]] (SCPs) can restrict the usage of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] at scale:

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyStepFunctionsCreation",
      "Effect": "Deny",
      "Action": [
        "states:CreateStateMachine"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringNotEqualsIgnoreCase": {
          "aws:PrincipalOrgID": "${ORG_ID}"
        }
      }
    }
  ]
}
```

**Performance & Reliability**

Throttling limits depend on the type of state machine. Standard state machines have a maximum concurrency of 1,000 executions per account per region. Express state machines offer up to 2,000 requests per second. To handle higher traffic, implement exponential backoff strategies with increasing delays between retries.

HA/DR patterns involve creating separate state machines in different regions, replicating data across them, and using routing mechanisms like [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include tagging state machines, activities, and executions. Tagged resources will appear in the [[billing|Cost Explorer]] tool, enabling better visibility and management. Calculation examples:

* Assuming $1.00 per million API calls, 100M calls would cost $100.
* Data passed between [[step-functions|states]] is billed at $0.00000025 per GB. A 5GB transfer would result in $0.00125 in charges.

**Professional Exam Scenarios**

Scenario 1: An organization wants to create a centralized [[vpc-flow-logs|logging]] solution using [[cloudwatch|CloudWatch Logs]], but they want to limit the creation of [[cloudwatch]] groups only to specific AWS accounts. Implement a solution using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

Correct answer: Create a centralized state machine in a master account that triggers [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] in member accounts to create [[cloudwatch]] groups. Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in member accounts to delegate permissions to the master account.

Incorrect answer: Directly create [[cloudwatch]] groups from the master account without using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. This approach does not follow the [[iam|best practices]] for cross-account access and governance.

Scenario 2: A company needs to process millions of images daily using [[lambda|AWS Lambda]], but some images require manual intervention before further processing. Propose a solution using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]], [[lambda]], and Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

Correct answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] to manage the overall workflow, including image upload to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], invoking a [[lambda]] function to process images, and triggering a human approval step if necessary.

Incorrect answer: Use [[lambda]] alone to process images and store metadata in [[dynamodb]]. This approach lacks the flexibility and modularity offered by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] and may become difficult to maintain over time.