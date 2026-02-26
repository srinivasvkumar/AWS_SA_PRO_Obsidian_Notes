**Advanced Architecture: [[eks]] [[Fargate]] Profiles**

[[eks]] [[Fargate]] Profiles simplify the deployment of serverless containers in Amazon [[eks]] by enabling the execution of Kubernetes pods without managing servers or clusters. The architecture is composed of several components:

* **[[Fargate]] Profile**: A [[Fargate]] Profile defines a set of rules that determine how your [[eks]] cluster should execute pods using [[Fargate]]. It includes information about the namespace, labels, and selectors to match pods.
* **Pod Execution Role**: This role grants [[Fargate]] permissions to interact with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] on behalf of the pod.
* **ENI Configuration**: [[Fargate]] uses Elastic Network Interfaces (ENIs) to provide network connectivity for pods. ENI configurations include [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], subnets, and [[appsync|security]] groups.

When working at scale, you can create multiple [[Fargate]] Profiles within a single [[eks]] cluster. Each profile can have its own settings for network configuration, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, and throttling limits.

To understand [[Fargate]] Profiles under the hood, it's important to know these key components:

* **AWS Firelens**: A log routing service used by [[Fargate]] to send container logs to various destinations like [[cloudwatch|CloudWatch Logs]], Fluent Bit, etc.
* **AWS App Mesh**: A service mesh offering from AWS that enables traffic management between microservices. [[Fargate]] Profiles support integration with App Mesh via sidecar containers.
* **Amazon ECR**: Container images must be stored in an ECR repository before they can be deployed as [[Fargate]] tasks.

**Comparison & Anti-Patterns**

| Service               | Suitable For                                                       | Not Suitable For                                            |
| --------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------ |
| [[eks]] [[Fargate]] Profiles   | Stateless applications requiring no infrastructure management<br>Applications needing rapid scaling<br>Developers who prefer managed services | Stateful applications requiring persistent storage<br>Applications with strict performance requirements<br>Workloads with specific OS-level dependencies |

Common design mistakes when using [[Fargate]] Profiles include:

* Assuming [[Fargate]] supports stateful workloads without considering the need for additional solutions such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] or Fargate-compatible databases.
* Ignoring the potential impact of networking [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] on application performance, such as [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and subnet selection.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[Fargate]] Profiles involve granting granular access permissions for each profile. An example JSON policy snippet might look like:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "ec2:CreateNetworkInterface",
              "ec2:DescribeNetworkInterfaces",
              "ec2:DeleteNetworkInterface"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access can be established through properly configured [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships. To enforce organization-wide restrictions, implement Service Control [[policies]] (SCPs) that limit the creation of [[Fargate]] resources based on tags, resource types, or specific principals.

**Performance & Reliability**

Throttling limits are crucial in maintaining system reliability. [[Fargate]] has configurable launch failures and concurrent executions limits per profile. In case of exceeding these limits, implement exponential backoff strategies to ensure predictable behavior.

HA/DR patterns for [[Fargate]] Profiles involve deploying multiple replicas across different availability zones or regions. Leverage [[Fargate]] Profiles' support for multi-region deployments and cross-region load balancing for enhanced [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost control for [[Fargate]] Profiles can be achieved by setting up [[Budgets]] and alerts based on specific namespaces or tags. Calculate costs using the AWS pricing calculator, taking into account factors such as vCPU and memory allocations, data transfer fees, and number of executions.

**Professional Exam Scenarios**

Scenario 1:
You need to implement a serverless solution for a stateless web application in AWS using [[eks]] [[Fargate]] Profiles. The application requires frequent scaling during peak hours. How would you optimize the solution for high availability while minimizing operational overhead?

Correct Answer:

* Create separate [[Fargate]] Profiles for development, staging, and production environments.
* Configure each environment to run in different Availability Zones.
* Implement Auto [[asg|Scaling policies]] based on CPU utilization or request count.
* Set up Application Load Balancer and configure [[route53|health checks]] for the application.
* Monitor application performance using Amazon [[cloudwatch]] metrics and alarms.

Incorrect Answer:

* Store session data directly on the [[Fargate]] instances.
* Use a single [[Fargate]] Profile for all environments.

Scenario 2:
Your team wants to deploy a containerized stateful application in an [[eks]] cluster using [[Fargate]] Profiles. However, the application requires persistent storage. What approach should you take to meet the storage requirement while using [[Fargate]] Profiles?

Correct Answer:

* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems to store shared data between containers.
* Mount the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] volume inside the container.
* Ensure proper network connectivity between the [[Fargate]] Profile and the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] mount point.

Incorrect Answer:

* Use local storage on [[Fargate]] instances.
* Attach [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes to [[Fargate]] containers.