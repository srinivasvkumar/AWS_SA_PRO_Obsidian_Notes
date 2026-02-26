**Advanced Architecture**

[[sam]] is an open-source framework that simplifies the deployment of serverless applications on AWS. It uses YAML templates to define resources, including [[lambda|AWS Lambda]] functions, Amazon [[api-gateway|API Gateway]] APIs, and Amazon [[dynamodb|DynamoDB tables]]. Under the hood, [[sam]] translates these templates into [[cloudformation]] [[cloudformation|stacks]] for provisioning.

At the core of [[sam]] is its ability to abstract away underlying infrastructure, allowing developers to focus on application logic. This abstraction extends beyond individual services, enabling powerful features like AWS::Serverless::Application, which packages multiple [[cloudformation|stacks]] as one deployable unit.

Global scaling in [[sam]] is achieved through native AWS services such as [[appsync|AWS AppSync]], which can be configured for multi-region deployments. However, it's essential to understand that not all services support automatic global distribution. In such cases, custom solutions using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] may be required.

**Comparison & Anti-Patterns**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[sam]]             | Simple serverless apps, [[cicd|CI/CD]] pipelines, development efficiency   |
| [[cloudformation]]   | Complex multi-account setups, infrastructure as code                |
| Terraform       | Multi-cloud environments, Immutable Infrastructure, version control |

Anti-patterns include using [[sam]] for managing non-serverless resources or overly complex architectures better suited for [[cloudformation]] or Terraform.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can leverage Service Control [[policies]] (SCPs) at the organization level to enforce [[control-tower|guardrails]]. For cross-account access, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with appropriate trust relationships should be created. Here's an example policy granting `lambda:InvokeFunction` permission:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:*:123456789012:function:MyFunction"
        }
    ]
}
```

**Performance & Reliability**

Throttling limits vary per service but generally follow [[iam|best practices]] around circuit breakers and exponential backoff strategies during failures. High availability/disaster recovery patterns often involve multi-region architectures, leveraging services like [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Latency-based routing]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented using AWS [[billing|Cost Explorer]] and tagging strategies. Calculating costs involves understanding billable events, invocations, and data transfer fees. Remember that [[sam]] itself does not directly impact costs; rather, it manages the deployment of cost-generating resources.

**Professional Exam Scenarios**

Scenario 1: A company wants to implement a multi-account strategy for their serverless application. How would you advise them to structure their accounts and resources?

Correct answer: They should create separate accounts for production, staging, testing, and development environments. Each account could have a dedicated [[sam]] template defining serverless resources. An Organization [[SCP]] could enforce centralized [[appsync|security]] [[policies]] across all accounts.

Incorrect answer: Using a single account for all environments would lead to poor resource separation and potential [[appsync|security]] risks.

Scenario 2: A high-traffic e-commerce site needs to ensure scalability and reliability while keeping costs minimal. What recommendations would you make regarding their serverless architecture?

Correct answer: Implement auto-scaling groups for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] based on traffic demands. Utilize Amazon [[api-gateway|API Gateway]]'s [[api-gateway|caching]] feature to reduce the load on backend systems. Enable AWS WAF rules for protecting web applications from common exploits.

Incorrect answer: Not implementing any form of [[api-gateway|caching]] might result in unnecessary strain on backend systems, leading to increased costs and potentially degraded performance during peak times.