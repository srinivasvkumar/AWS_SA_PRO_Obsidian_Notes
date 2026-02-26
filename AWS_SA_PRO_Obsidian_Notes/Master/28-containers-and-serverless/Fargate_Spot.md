### Advanced Architecture
At its core, [[Fargate]] Spot is a serverless compute engine for containers that utilizes spare capacity in the Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] and Amazon Elastic Kubernetes Service ([[eks]]) clusters. It offers significant cost savings compared to [[Fargate]] and [[Fargate]] Dedicated by utilizing unused [[ec2]] instances in the Spot market.

Internally, [[Fargate]] Spot uses a cluster management system that provisions and de-provisions resources dynamically based on demand and available capacity. The system supports multiple instance types and Availability Zones within a region, allowing it to scale globally while maintaining high availability.

When launching tasks or services using [[Fargate]] Spot, you can specify a tolerance level for interruptions. This value ranges from `LOW` to `HIGH`, indicating how often your tasks may be interrupted due to changes in spot prices or capacity. If your workload requires minimal disruption, you should use [[Fargate]] instead of [[Fargate]] Spot.

### Comparison & Anti-Patterns

| Feature                         | [[Fargate]] Spot       | [[Fargate]]             | ECS/EKS Instances   |
| ------------------------------- | ------------------ | ------------------- | ------------------ |
| Cost                            | Lowest             | Higher              | Low                |
| Interruption Tolerance          | High, Medium, Low  | None               | N/A               |
| Instance Type Selection         | Automatic          | Manual              | User-defined      |
| Scaling Mechanism              | Cluster Management | Task/Service        | Auto Scaling Group |
| Multi-Account Strategies         | Limited            | Full Support        | Full Support       |

Anti-patterns include:

* Running critical batch jobs on [[Fargate]] Spot without proper error handling and retries
* Using [[Fargate]] Spot for mission-critical applications requiring high availability and low latency
* Utilizing [[Fargate]] Spot for long-running tasks with strict completion requirements

### [[appsync|Security]] & Governance

To secure [[Fargate]] Spot environments, follow these [[iam|best practices]]:

1. Implement fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON snippets like the following example, which grants permissions to create and manage [[Fargate]] Spot tasks and services:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:RunTask",
                "ecs:DescribeTasks"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEqualsIfExists": {
                    "aws:RequestTag/cluster": "${aws_ecs_cluster.this.name}"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "eks:DescribeNodegroup",
                "eks:ListUpdates",
                "eks:DescribeUpdate",
                "eks:CreateAddon",
                "eks:DeleteAddon"
            ],
            "Resource": "*"
        }
    ]
}
```
2. Enable cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]
3. Implement Organization Service Control [[policies]] (SCPs) to restrict [[Fargate]] Spot usage across accounts

### Performance & Reliability

[[Fargate]] Spot has throttling limits and exponential backoff strategies similar to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]. To ensure performance and reliability, consider these guidelines:

1. Monitor task and service statuses using [[cloudwatch|CloudWatch alarms]] and notifications
2. Implement an exponential backoff strategy when encountering [[api-gateway|errors]] during task creation or updates
3. Design for high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] using multi-region deployments and data replication

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] techniques for [[Fargate]] Spot include:

1. Analyzing historical price trends and selecting optimal launch configurations based on the desired interruption tolerance
2. Setting up [[billing]] alarms and [[Budgets]] to monitor costs and avoid unexpected charges
3. Leveraging [[cost-allocation-tags|cost allocation tags]] to track spending and identify areas for potential cost reduction

### Professional Exam Scenarios

#### Scenario 1:
An e-commerce company needs to run a batch job that processes customer orders daily. They expect the job to take around 8 hours to complete but want to minimize costs. Which solution would you recommend?

Correct Answer: Utilize [[Fargate]] Spot with a high interruption tolerance level and implement error handling and retries for the batch job.

Incorrect Answer: Use [[Fargate]] as it does not support interruption tolerance and will result in higher costs than [[Fargate]] Spot.

#### Scenario 2:
A media organization wants to process video streams using containerized microservices. They require high availability and low latency. Should they use [[Fargate]] Spot?

Correct Answer: No, [[Fargate]] Spot is not suitable for this scenario because it cannot guarantee the required high availability and low latency. Instead, they should use [[Fargate]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|dedicated instances]] based on their specific requirements.

Incorrect Answer: Yes, [[Fargate]] Spot provides sufficient high availability and low latency for the media organization's needs.