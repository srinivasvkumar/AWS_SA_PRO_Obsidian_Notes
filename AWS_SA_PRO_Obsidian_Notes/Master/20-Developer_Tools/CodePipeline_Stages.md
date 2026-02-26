### Advanced Architecture
At its core, [[cicd|AWS CodePipeline]] is a continuous integration and continuous delivery ([[cicd|CI/CD]]) service that automates the build, test, and deployment of applications. It consists of [[api-gateway|stages]] and actions, where each stage contains one or more actions. A key aspect of CodePipeline is the ability to model and execute complex deployments across multiple accounts and regions. This can be achieved using cross-account and cross-region deployments, as well as leveraging [[organizations|AWS Organizations]] for centralized management and enforcement of [[policies]].

Internally, CodePipeline uses [[step-functions|AWS Step Functions]], which provides the underlying state machine functionality, enabling the service to manage the execution of individual actions within a pipeline. Each action represents an activity in the application development lifecycle, such as building source code or deploying to a test environment.

### Comparison & Anti-Patterns

| Service      | Use Cases                                                                    |
| ------------ | ---------------------------------------------------------------------------- |
| CodeBuild    | Primarily used for building and testing code within a pipeline               |
| CodeDeploy   | Typically employed for in-place deployments or blue/green deployments        |
| CodePipeline | Ideal for orchestrating workflows and managing dependencies between services |

Anti-patterns include using CodePipeline when simple, linear workflows suffice, where CodeBuild and CodeDeploy alone would be sufficient. Another common mistake is not properly defining failure [[cloudformation|conditions]] and error handling, leading to potential data loss or system downtime.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] often involve allowing specific users or roles to invoke certain actions within a pipeline. For instance, you might grant permissions to invoke a [[lambda]] function responsible for triggering a pipeline:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": [
                "codepipeline:StartPipelineExecution"
            ],
            "Resource": "arn:aws:codepipeline:us-east-1:123456789012:pipeline/my-pipeline",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:lambda:us-east-1:123456789012:function:my-lambda-function"
                }
            }
        }
    ]
}
```
Cross-account access can be established by configuring the appropriate resource-based [[policies]] on the source resources and pipelines. Additionally, [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) can be utilized to enforce restrictions and [[iam|best practices]] across member accounts.

### Performance & Reliability

CodePipeline has throttling limits based on the number of pipelines per region and the number of pipeline executions per account. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies during pipeline execution retries. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns should incorporate multi-region pipelines, ensuring minimal disruption during regional outages.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

To optimize costs, enable CodePipeline's built-in cost controls, such as stopping pipelines when they aren't actively being used. Also, leverage granular cost controls like specifying the exact regions and resources required for each action. Calculation examples might involve determining the cost difference between running pipelines in different regions or with varying levels of parallelism.

### Professional Exam Scenario

**Scenario 1:** Your organization requires a [[cicd|CI/CD]] solution capable of supporting multiple AWS accounts and regions. The solution must also provide fine-grained control over pipeline execution and minimize costs.

*Correct answer:* Implement a multi-account, multi-region CodePipeline solution with separate pipelines for each environment (dev, staging, prod). Leverage [[organizations|AWS Organizations]] and SCPs to enforce [[policies]] and restrict access. Utilize AWS Resource Access Manager to share necessary resources between accounts. Implement custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] to enable cross-account access and limit actions to specific users. Configure each pipeline to utilize the most cost-effective regions and resources.

*Incorrect answer:* Use a single CodePipeline instance with cross-region replication, as it does not offer fine-grained control over pipeline execution and may lead to increased costs due to cross-region data transfer.

**Scenario 2:** Your team needs to integrate a new open-source project into your existing CodePipeline without modifying the original repository.

*Correct answer:* Create a webhook from CodePipeline to a custom event bus in Amazon [[eventbridge]]. Set up a rule to listen for events from the webhook and trigger an [[lambda|AWS Lambda]] function. Instruct the [[lambda]] function to clone the open-source repository and perform any necessary modifications before invoking the CodePipeline.

*Incorrect answer:* Modify the original open-source repository directly within CodePipeline, as it violates open-source principles and could introduce compatibility issues.