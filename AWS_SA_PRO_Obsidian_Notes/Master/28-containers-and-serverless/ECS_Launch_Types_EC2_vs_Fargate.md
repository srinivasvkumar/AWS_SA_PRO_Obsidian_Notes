Advanced Architecture
---------------------

### [[ecs]] [[ec2]] Launch Type

The [[ec2]] launch type in Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] uses the Amazon [[ec2]] infrastructure to run containers. Each [[ecs]] task is mapped to an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Auto Scaling]] group of [[ec2]] instances. The [[ec2]] instances can be launched within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and can span across multiple Availability Zones (AZs) for high availability. To achieve horizontal scaling and load balancing, the Application Load Balancer or Network Load Balancer can be used along with the [[ec2]] launch type.

For global scale applications, you can use the [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to improve latency by directing user traffic through the optimal AWS edge location. This allows your containerized applications to have a consistent and responsive user experience worldwide.

### [[ecs]] [[Fargate]] Launch Type

The [[Fargate]] launch type eliminates the need to manage the underlying infrastructure as it removes the responsibility of provisioning and managing servers. It is designed to work seamlessly with services like Amazon ECR, AWS App Mesh, AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]], and AWS [[appsync|Security]] Groups. With [[Fargate]], you specify the CPU and memory requirements for each container instance, and the platform manages the allocation of resources accordingly.

Comparison & Anti-Patterns
---------------------------

| Criteria | [[ecs]] [[ec2]] | [[ecs]] [[Fargate]] |
| --- | --- | --- |
| Resource management | Requires manual configuration of [[ec2]] instances | Automated resource management |
| Scalability | Horizontal scaling using ASGs and load balancers | Implicitly scalable based on specified resources |
| Cost | Lower costs when tasks are long-running | Higher costs due to pay-per-use model |
| Observability | Integrates with Prometheus, Datadog, etc. | Limited built-in monitoring capabilities |
| Operational overhead | More operational overhead due to server maintenance | Less operational overhead due to automated resource management |

Anti-patterns include using [[ecs]] [[ec2]] for short-lived tasks that require quick scaling and frequent deployments, as well as using [[ecs]] [[Fargate]] for long-running tasks that would result in higher costs.

[[appsync|Security]] & Governance
----------------------

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[ecs]] [[ec2]] involve granting permissions to various components like [[ec2]] instances, Task Definition roles, and [[ecs]] services. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:RunTask"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ecs:cluster": "${aws_ecs_cluster.example.name}"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceTypes",
                "ec2:DescribeRegions",
                "ec2:DescribeAvailabilityZones"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access between [[ecs]] [[ec2]] and other accounts requires creating a role in the source account, attaching necessary [[policies]], and assuming that role from the destination account. An organization [[appsync|Security]] Control Policy ([[SCP]]) can enforce restrictions on specific actions and resources.

Performance & Reliability
--------------------------

Throttling limits vary depending on the launch type. For [[ecs]] [[ec2]], there are [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] related to API calls, concurrent tasks, and task counts per cluster. For [[ecs]] [[Fargate]], throttling limits include the number of concurrent tasks, task network bandwidth, and task CPU and memory usage. Implementing exponential backoff strategies can help mitigate issues caused by throttling.

HA/DR patterns include running multiple replicas of [[ecs]] clusters in different AZs and enabling task definition revision history. Additionally, using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] with [[route53|health checks]] and failover configurations can ensure high availability.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls for [[ecs]] [[ec2]] involve setting up budget alerts, configuring [[billing]] tags, and reserving capacity for predictable workloads. In contrast, [[ecs]] [[Fargate]] charges based on the actual compute resources consumed by the application, so granular cost controls mainly focus on optimizing resource utilization.

Professional Exam Scenario Questions
------------------------------------

Question 1: Your company has a containerized application that runs continuous integration tests. Due to the unpredictable nature of these tests, the team needs to scale quickly during peak times. Which launch type would best suit their needs?

Correct answer: [[ecs]] [[Fargate]]. Since the CI tests are unpredictable and may not last long, using [[Fargate]] ensures quick scaling without having to worry about managing infrastructure.

Incorrect answer: [[ecs]] [[ec2]]. While [[ecs]] [[ec2]] provides more control over the underlying infrastructure, its manual scaling might not meet the dynamic requirements of the CI tests.

Question 2: Your organization wants to deploy a containerized web application globally while minimizing latency. They want to leverage Amazon [[ecs]] but are unsure if they should choose the [[ec2]] or [[Fargate]] launch type. What recommendations would you provide?

Correct answer: Both [[ec2]] and [[Fargate]] can be used with [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to minimize latency. However, since the question does not mention any preference towards managing infrastructure, either option could be chosen based on the organization's expertise.

Incorrect answer: Choosing one launch type over another solely based on latency reduction is incorrect because both options support [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]].