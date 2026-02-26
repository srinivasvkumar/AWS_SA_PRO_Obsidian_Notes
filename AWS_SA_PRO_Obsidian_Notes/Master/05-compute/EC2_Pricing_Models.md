## Advanced Architecture
At its core, [[ec2]] provides virtualized computing environments known as instances. Instances can be categorized into various instance types based on their underlying hardware resources such as CPU, memory, and networking capabilities. To ensure high availability (HA) and fault tolerance (FT), [[ec2]] offers regional and zonal deployment options. For global applications, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] can be used in conjunction with [[route53|health checks]] and DNS failover [[policies]] to distribute traffic across multiple regions.

### Under the Hood
Each [[ec2]] instance runs on top of the Xen hypervisor, enabling multiple virtual machines to share a single physical host while maintaining isolation. The primary abstraction layer is called the "nested virtualization" which allows running guest operating systems within the virtual environment. Another critical component is the Amazon Elastic Network Adapter (ENA) that ensures efficient network communication between instances, regardless of their location or connectivity requirements.

## Comparison & Anti-Patterns
While [[ec2]] provides significant flexibility and scalability, it may not always be the best fit for every workload. Here are some comparisons and anti-patterns:

| Service          | Use Cases                                                              |
| ---------------- | --------------------------------------------------------------------- |
| [[lambda]]          | Event-driven, serverless compute for mobile/web apps, [[iot]], etc.    |
| [[Fargate]]         | Serverless container management without managing infrastructure   |
| Batch           | Scheduled, parallel, and distributed batch processing                |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]]      | Simplified container deployment and scaling for web applications     |

Anti-patterns include deploying stateful services like databases on [[ec2]] without proper backups, using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] for latency-sensitive workloads, and relying solely on [[ec2]] for load balancing instead of leveraging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like [[ALB]] or [[NLB]].

## [[appsync|Security]] & Governance
Implementing granular [[appsync|security]] [[policies]] requires a combination of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, permissions, cross-account access, and organization Service Control [[policies]] (SCPs):

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy Example
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

### Cross-Account Access
To grant cross-account access, you need to create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and attach a policy allowing the target account principal to perform specific actions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::TARGET_ACCOUNT_ID:root"
            },
            "Action": [
                "ec2:DescribeInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

### Organization SCPs
Service Control [[policies]] (SCPs) restrict or allow specific API operations at the organizational level. An example denying the creation of new [[ec2]] instances would look like:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "ec2:RunInstances"
            ],
            "Resource": "*"
        }
    ]
}
```

## Performance & Reliability
To achieve optimal performance and reliability, consider implementing throttling limits, exponential backoff strategies, and high availability/disaster recovery patterns:

### Throttling Limits
Throttling limits help prevent overloading the system by enforcing maximum request rates per API operation. For example, the default limit for DescribeInstances is 50 requests per second. If this limit is exceeded, the API will return a `400 SlowDown` error response.

### Exponential Backoff Strategies
Exponential backoff strategies should be implemented when handling transient [[api-gateway|errors]] due to rate limiting or service unavailability. This technique involves gradually increasing the waiting time between retries after each consecutive failure.

### High Availability/Disaster Recovery Patterns
Deploy [[ec2]] instances across multiple availability zones (AZs) within a region to achieve high availability. Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns by launching standby instances in another region and synchronizing data using services like [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or AWS Database Migration Service.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be achieved through tagging resources, configuring budget alerts, and utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|reserved instances]] (RIs) or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]]:

### Tagging Resources
Tagging enables tracking costs at different levels of your application stack. For example, tags can represent departments, teams, or environments.

### Budget Alerts
Configure budget alerts to notify stakeholders when spending thresholds are met or exceeded.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs)
[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved instances]] provide upfront discounts for long-term workloads. Consider converting frequently utilized instance types to RIs to save on costs.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]
[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]] offer significant savings compared to on-demand instances but come with the risk of being terminated if the market price increases above your maximum bid. Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] for workloads that can tolerate interruptions, such as background tasks or test/dev environments.

## Professional Exam Scenarios

### Scenario 1
Your company operates a mission-critical application that requires high availability and low-latency access to data stored in Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]. How would you design a solution that meets these requirements?

#### Correct Answer
Use [[ec2]] instances deployed across multiple AZs with Multi-AZ [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] setup. Implement read replicas for improved read performance and use [[elasticache]] for [[api-gateway|caching]] frequent database queries. Configure route tables and [[appsync|security]] groups to enable private connectivity between [[ec2]] instances and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances.

#### Incorrect Answers
1. Deploy all components in a single AZ: This approach lacks high availability and introduces potential downtime during AZ failures.
2. Use on-premises servers connected via [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]: While this solution maintains low-latency access, it increases complexity and reduces reliability due to potential WAN issues.

### Scenario 2
Your organization has strict governance [[policies]] requiring individual departmental chargebacks. How would you implement a cost allocation strategy using [[ec2]]?

#### Correct Answer
Create a naming convention for [[ec2]] instances and associated resources. Apply relevant tags to each resource representing departments, teams, or environments. Enable [[billing|Cost Explorer]] and set up budget alerts for each department.

#### Incorrect Answer
1. Use separate AWS accounts for each department: This approach increases complexity and makes centralized management more challenging.
2. Manually track usage: Relying on manual methods introduces human error and decreases accuracy.