Advanced Architecture
---------------------

At the professional level, Docker on AWS can be orchestrated using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] or Amazon Elastic Kubernetes Service ([[eks]]). Both services operate at the infrastructure level and provide advanced features such as automatic scaling, rolling updates, and self-healing capabilities.

### [[ecs]] [[RDS_Instance_Types|Internals]]

[[ecs]] uses the following components to orchestrate containers:

* **Clusters**: A logical grouping of container instances.
* **Task Definitions**: A JSON file that describes one or more containers, including their Docker images, CPU and memory requirements, port [[cloudformation|mappings]], and environment variables.
* **Services**: An [[ecs]] component that maintains a specified number of tasks in a cluster. Services support load balancing and rolling updates.
* **Container Instances**: [[ec2]] instances that have the [[ecs]] agent installed. The agent communicates with the [[ecs]] service to start, stop, and maintain tasks.

### [[RDS_Instance_Types|Global Scale Considerations]]

Global scale can be achieved by deploying multiple [[ecs]] clusters across different regions. Each cluster can be managed independently and serve a specific geographical area. To distribute traffic between these clusters, use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] with Geo DNS routing. This allows you to create [[route53|health checks]] for each cluster and define failover [[cloudformation|conditions]] based on the health check status.

### Under the Hood Mechanics

When creating an [[ecs]] task definition, specify the Docker image, CPU and memory requirements, and other settings. [[ecs]] then creates a task that runs on a container instance. If the task requires more resources than what's available on a single container instance, it will span multiple instances. This is known as a "task placement strategy" and can be configured during task definition creation.

Comparison & Anti-Patterns
---------------------------

| Service | Use Case |
| --- | --- |
| [[ecs]] | Best suited for applications built using the microservices architecture pattern, with multiple containers per application. It provides tight integration with [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like Application Load Balancer, [[Fargate]], and [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]]. |
| [[eks]] | Ideal when migrating existing Kubernetes workloads to AWS or when working with third-party tools that require Kubernetes compatibility. It offers greater flexibility but may introduce additional management complexity compared to [[ecs]]. |
| [[lambda]] | Suitable for event-driven, serverless architectures. Not recommended for long-running processes or stateful applications. |

Common anti-patterns include using [[ecs]] or [[eks]] for monolithic applications, which could benefit from alternative services like [[ec2]] or [[lambda]]. Overusing [[ecs]] or [[eks]] services without considering the actual needs of the application can lead to increased costs and operational overhead.

[[appsync|Security]] & Governance
----------------------

To enforce [[appsync|security]] and governance [[iam|best practices]], consider implementing the following:

* Create separate [[ecs]] clusters or [[eks]] namespaces for production, staging, and development environments.
* Implement granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON snippets. For example, grant permissions only to specific actions, resources, and [[cloudformation|conditions]].
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeClusters",
                "ecs:ListTasks",
                "ecs:RegisterTaskDefinition"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "vpc-12345678"
                }
            }
        }
    ]
}
```
* Enable cross-account access by creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and assuming it from the destination account. In the destination account, create a policy that grants necessary permissions to the assumed role.
* Use [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to centrally manage and enforce [[appsync|security]] [[policies]] across multiple accounts.

Performance & Reliability
--------------------------

To ensure high performance and reliability, consider the following:

* Monitor throttling limits for [[ecs]] and [[eks]] services. For example, [[ecs]] has a limit of 500 task definitions per region. If your usage approaches this limit, request a limit increase.
* Implement exponential backoff strategies for retries. This helps prevent your application from overwhelming the [[ecs]] or [[eks]] API with too many requests.
* Design for High Availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] ([[dr]]) by deploying multiple [[ecs]] clusters or [[eks]] namespaces across different regions. Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] with Geo DNS routing for automatic failover.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
-----------------

Granular cost controls can be implemented through the following methods:

* Set up [[billing]] alarms to monitor monthly spending. This can help identify unexpected charges early.
* Use AWS [[Fargate]] instead of [[ec2]] instances whenever possible. [[Fargate]] pricing is based on vCPU and memory usage, so you only pay for the resources consumed by your containers.
* Configure [[ecs]] cluster capacity providers to automatically scale the number of container instances based on demand. This ensures efficient resource utilization while minimizing idle capacity.

Professional Exam Scenarios
----------------------------

Scenario 1:

An e-commerce company wants to deploy a new application using [[ecs]]. They expect high traffic during holiday seasons and need a solution that scales automatically. They also want to minimize costs during off-peak hours.

Correct Answer: Use [[ecs]] with [[Fargate]] and set up autoscaling rules based on CPU utilization.

Incorrect Answer: Use [[ec2]] instances with fixed capacity.

Justification: Using [[Fargate]] eliminates the need to manage the underlying infrastructure, allowing the company to focus on building the application. Autoscaling rules ensure the application scales based on demand, reducing costs during off-peak hours.

Scenario 2:

A software development firm wants to migrate its existing Kubernetes workloads to AWS. They need to implement fine-grained [[appsync|security]] [[policies]] and control access to Kubernetes resources.

Correct Answer: Use [[eks]] with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and Service Control [[policies]] (SCPs).

Incorrect Answer: Use [[ecs]] with generic [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Justification: [[eks]] provides better compatibility with Kubernetes, making it easier to migrate existing workloads. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and SCPs allow the firm to implement fine-grained [[appsync|security]] [[policies]] and control access to Kubernetes resources.