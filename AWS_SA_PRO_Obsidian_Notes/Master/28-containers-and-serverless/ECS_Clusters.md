**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[ecs]] Clusters operate on the [[ec2]] platform and can be configured in multiple ways based on workload requirements. The fundamental components include the [[ecs]] Agent, Task Definition, Task, Service, and Cluster. An [[ecs]] Cluster is a logical grouping of tasks and services.

*Task Definitions* specify container configurations such as CPU, memory, port [[cloudformation|mappings]], environment variables, and Dockerfile paths. They also define entry point commands and container dependencies.

*Services* maintain a specified state within an [[ecs]] Cluster by launching or terminating tasks. Services can be load balanced using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] or Application Load Balancer.

*Task Scheduling Strategies* support Long Run Draining, which enables graceful shutdown of tasks while new ones start. Services can use the DAEMON or REPLICA scheduler strategy. The DAEMON strategy ensures that one instance of the task runs on each registered [[ec2]] instance. The REPLICA strategy maintains a specified number of tasks in a cluster.

*Capacity Providers* allow integration with [[Fargate]], [[ec2]], or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]]. [[ecs]] Clusters can span multiple availability zones and regions through *Global/Regional Clusters*. Global Clusters enable cross-region capacity provisioning with automatic failover and regional [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
|---|---|
| [[ecs]] | Containerized applications requiring fine-grained control over infrastructure. |
| [[Fargate]] | Serverless compute for containers without managing underlying instances. |
| [[lambda]] | Event-driven functions without managing servers or containers. |
| Batch | Job scheduling and processing for large-scale batch computing. |

Anti-patterns include running stateless web applications directly on [[ecs]] instead of ALB-backed services, or using [[ecs]] when [[Fargate]] would provide better resource utilization and lower operational overhead.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow least privilege principles. For example, restricting API calls to specific resources:
```json
{
    "Effect": "Allow",
    "Action": [
        "ecs:DescribeClusters",
        "ecs:ListTasks"
    ],
    "Resource": [
        "*"
    ],
    "Condition": {
        "StringEquals": {
            "ecs:cluster": [
                "arn:aws:ecs:us-west-2:123456789012:cluster/production-cluster"
            ]
        }
    }
}
```
Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships. Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] like disallowing direct [[ecs]] API usage.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[ecs]] Clusters have throttling limits on API calls. Exponential backoff strategies help manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] during high request rates. High Availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] patterns involve deploying services across multiple AZs, enabling Multi-Region replication, and configuring Failover settings.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include selecting appropriate Capacity Providers, optimizing Task Memory and CPU allocations, and utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]. AWS [[billing|Cost Explorer]] provides detailed cost analysis and recommendations.

**6. Professional Exam Scenario 1**

An organization has two AWS accounts: `prod` and `non-prod`. Both share a common [[AWS Organization]] root account. They want to deploy a highly available [[ecs]] application with separate production and staging environments, but no direct communication between them.

Correct Answer: Create two [[ecs]] clusters, one per account, with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering and [[appsync|Security]] Groups controlling traffic flow. This approach avoids direct communication between environments while maintaining separation.

Incorrect Answer: Using a single [[ecs]] cluster with different Task Definitions does not ensure complete isolation between environments.

**Professional Exam Scenario 2**

An e-commerce company needs to process orders from their website and update inventory levels in real-time. They expect varying order volumes throughout the day.

Correct Answer: Deploy the solution using [[ecs]] and [[Fargate]], scaling Tasks dynamically based on demand. Implement event-driven architecture with [[lambda]] for real-time inventory updates.

Incorrect Answer: Running the entire solution on [[ecs]] may result in underutilization of resources during low demand periods.