**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[ecs]] Services is a core component of Amazon [[ecs]], providing task definition management, load balancing, and service discovery features. The [[RDS_Instance_Types|internals]] involve [[ecs]] Agent, Task Definition, Service, and Cluster. The [[ecs]] Agent runs on each [[ec2]] instance or container instance and sends information about the tasks running on the instance to the [[ecs]] service API. The Task Definition describes one or more containers, up to a maximum of ten, that form your application. A Service uses a task definition to maintain a specified number of instances of your application. An [[ecs]] Cluster consists of one or more services, and it can span multiple Availability Zones within an region.

Global scaling is achieved through the AWS Global Infrastructure, which allows you to deploy applications across multiple regions. To distribute traffic between regions, you can use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] DNS failover or Geo DNS routing [[policies]]. For cross-region replication, you can use [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]], [[Storage Gateway]], or self-managed solutions like rclone.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Feature | [[ecs]] Services | [[Fargate]] | [[lambda]] | [[eks]] |
|---|---|---|---|---|
| Container Orchestration | Yes | Yes | Limited | Kubernetes |
| Serverless | No | Yes | Yes | No |
| Scalability | Excellent | Excellent | Excellent | Good |
| Cost Model | Pay per instance-hour | Pay per vCPU & memory-hour | Pay per invocation time | Pay per node instance-hour |

Anti-patterns include using [[ecs]] Services when [[Fargate]] would be sufficient or preferable due to reduced operational burden. Another anti-pattern is attempting to run stateful workloads in [[ecs]] without proper data persistence and backup strategies.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeClusters",
                "ecs:ListTasks"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-account access requires creating a role in the source account with the necessary permissions, then assuming that role from the destination account. Organizational Service Control [[policies]] (SCPs) may limit resource usage by restricting actions on specific resources.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the type of [[ecs]] service used. Service quotas can be increased via support tickets. Exponential backoff strategies should follow [[iam|best practices]], such as starting at 2x the base wait time and doubling until success or reaching a predefined maximum wait time.

HA/DR patterns include deploying services across multiple AZs, using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] for service discovery, and enabling AWS Application Auto Scaling. For [[dr]], deploy to another region and enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] or geography-based routing.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for non-critical workloads, rightsizing [[ec2]] instances based on task requirements, and leveraging Savings Plans. Calculating costs involves understanding the pricing model, including the hourly rate for [[ec2]] instances and additional charges for data transfer and other resources.

**6. Professional Exam Scenario Questions**

*Scenario 1:* Your company wants to deploy a web application globally using [[ecs]] Services, but needs to minimize latency for end-users. What architecture should you implement?

*Correct Answer:* Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] DNS failover or Geo DNS routing [[policies]] to distribute traffic between multiple [[ecs]] clusters in different regions. Ensure each cluster has its own [[ALB]], target group, and [[ecs]] service. This setup provides low-latency connections to users worldwide.

*Incorrect Answer:* Deploy the entire application in a single [[ecs]] cluster. While this approach simplifies management, it introduces unnecessary latency for end-users located far away from the cluster.

*Scenario 2:* Your organization uses [[ecs]] Services for containerized microservices and wants to ensure they cannot exceed their monthly budget. How should you address this concern?

*Correct Answer:* Implement [[billing|AWS Budgets]] to track [[ecs]] spending against custom thresholds. Enable alerts for budget deviation. Additionally, use AWS [[billing|Cost Explorer]] to visualize costs and identify potential savings opportunities.

*Incorrect Answer:* Rely solely on setting [[ecs]] Service quotas. Although these quotas help control resource utilization, they do not directly limit spending.