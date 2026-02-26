**Advanced Architecture**

[[Fargate]] is a serverless compute engine for containers that works with both Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] and Amazon Elastic Kubernetes Service ([[eks]]). [[Fargate]] makes it easy for you to focus on building your applications, as it removes the need to provision and manage servers.

Internally, [[Fargate]] uses the following components:

- ** Jenkins X (for [[cicd|CI/CD]])
- ** AWS Firelens (for [[vpc-flow-logs|logging]] configuration)
- ** [[ecs]] Agent / Kubelet (for task execution)
- ** Nitro System (to enforce instance-level [[appsync|security]])

Global scale is achieved by deploying tasks across multiple Availability Zones within a region. To orchestrate these tasks, [[Fargate]] utilizes task networking, which includes an ENI (Elastic Network Interface) per container, allowing communication between tasks in the same task definition revision and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

**Comparison & Anti-Patterns**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[Fargate]]         | Microservices, batch processing, lift-and-shift workloads           |
| [[lambda]]         | Event-driven, small code bases, real-time file processing            |
| [[ec2]]             | Large monolithic applications, customizable infrastructure        |
| Batch            | Data-intensive workloads, long-running jobs, parallel processing    |

Anti-patterns include using [[Fargate]] for stateful workloads like databases, as well as running short-lived stateless tasks on [[ec2]] instances.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON snippets such as:

```json
{
    "Effect": "Allow",
    "Action": ["ecs:RunTask"],
    "Resource": "*",
    "Condition": {
        "StringEquals": {
            "aws:RequestedServiceName": [
                "arn:aws:ecs:us-west-2:123456789012:service/my-cluster/my-service"
            ]
        }
    }
}
```

Cross-account access can be established through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]], while Organization Service Control [[policies]] (SCPs) can help set centralized [[control-tower|guardrails]] for [[Fargate]] usage.

**Performance & Reliability**

Throttling limits in [[Fargate]] depend on the task type and number of vCPUs:

- [[Fargate]] (1vCPU): 64 tasks per cluster per AZ
- [[Fargate_Spot]] (1vCPU): 25% of [[Fargate]] limit
- [[Fargate]] (2vCPU): 32 tasks per cluster per AZ
- [[Fargate_Spot]] (2vCPU): 25% of [[Fargate]] limit

Exponential backoff strategies should be applied when handling throttling [[api-gateway|errors]], using techniques like Jitter and Token Bucket algorithms.

HA/DR patterns involve placing tasks in different AZs and regions, utilizing services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and Global Accelerator for load distribution.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls in [[Fargate]] include selecting specific task types (e.g., [[Fargate]] or [[Fargate]]\_SPOT) and configuring launch types (e.g., PROVISIONED or UNRESERVED). Moreover, rightsizing [[Fargate]] tasks based on CPU and memory requirements leads to optimal pricing.

Calculation example:

- Task count: 10 ([[Fargate]]\_SPOT, 2vCPU)
- Duration: 1 month
- Region: US West (Oregon)
- Total cost: (10 tasks \* $0.04048 per hour \* 24 hours \* 30 days) + data transfer fees = ~$67.8

**Professional Exam Scenarios**

Scenario 1:
You are working for a company that needs to process large volumes of data in batches. The company has strict SLAs, and requires high availability. Which architecture would best suit their needs?

Correct answer: [[Fargate]] with [[ecs]], using a multi-AZ deployment and batch jobs.

Incorrect answer: Using [[lambda]] for batch processing is not ideal due to its event-driven nature and potential cold start issues.

Scenario 2:
Your organization wants to implement a multi-account strategy for [[Fargate]] usage, ensuring compliance with corporate [[policies]]. How would you achieve this?

Correct answer: Create an [[AWS Organization]], apply SCPs to restrict [[Fargate]] actions, and enable cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

Incorrect answer: Allowing unrestricted access to [[Fargate]] without proper governance and monitoring could lead to [[appsync|security]] risks and unexpected costs.