## [[RDS_Instance_Types|1. Advanced Architecture]]

Auto Scaling Groups (ASGs) are central to managing scalable and resilient compute resources in AWS. At their core, ASGs rely on three key components: Launch Configurations, Launch Templates, and the Auto Scaling service itself.

### Launch Configurations

A Launch Configuration defines the instance type(s), AMI ID(s), [[appsync|security]] group(s), and block device mapping(s) to be used when launching new instances. They are immutable and can only be associated with a single [[asg]].

### Launch Templates

Launched in 2018, Launch Templates provide a more flexible and versioned alternative to Launch Configurations. They support various parameters such as the instance type, AMI ID, [[appsync|security]] groups, key pair, and block devices. Additionally, Launch Templates enable versioning, allowing you to create new versions without affecting running instances.

### Under the Hood Mechanics

When an [[asg]] launches instances, it creates a logical grouping of these instances using Amazon [[ec2]] instance metadata called `aws:autoscaling:group`. This metadata enables advanced features like [[route53|health checks]], [[asg|scaling policies]], and [[asg|lifecycle hooks]].

At the global level, Amazon offers Application Auto Scaling (App AS) which allows scaling management across multiple AWS services, including [[ec2]], [[ecs]], [[emr]], [[dynamodb]], and Spot Fleet. App AS uses [[cloudwatch|CloudWatch alarms]] to trigger scaling actions based on predefined metrics.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service           | Use Case                                                              |
| ----------------- | -------------------------------------------------------------------- |
| **[[asg]]**           | Vertical / horizontal scaling, rolling updates, patching            |
| **[[ecs]] Clusters**   | Containerized applications requiring orchestration                    |
| **Spot Fleet**    | Cost-optimized workloads utilizing spare capacity                     |
| **Batch**         | Parallel, scheduled, and ad-hoc batch processing                      |
| **[[Lambda@Edge]]**  | Serverless, edge-computing use cases (e.g., image optimization, redirection) |

Anti-patterns include:

- Using ASGs for container orchestration instead of ECS/Fargate
- Running stateless applications in ASGs without proper load balancing
- Implementing custom scaling solutions instead of using built-in App AS

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve permissions for creating/updating ASGs, controlling instance termination, and attaching/detaching network interfaces. Here's an example JSON policy snippet:

```json
{
  "Effect": "Allow",
  "Action": [
    "autoscaling:CreateAutoScalingGroup",
    "autoscaling:UpdateAutoScalingGroup"
  ],
  "Resource": "*"
}
```

Cross-account access is possible through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with trust relationships between accounts. For stricter governance, Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] at the organization level.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits vary depending on the region and API call rate. To avoid [[api-gateway|errors]] due to throttling, implement exponential backoff strategies using the AWS SDKs.

HA/DR patterns often involve placing ASGs in different regions or availability zones, ensuring that application endpoints have redundancy and failover capabilities.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include setting per-unit costs based on instance types and purchase options (On-Demand, Reserved, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]]). Calculation examples:

- On-Demand: $0.12/hour * 10 instances = $1.20/hour
- Reserved: $0.07/hour * 10 instances = $0.70/hour
- Spot: $0.02/hour * 10 instances = $0.20/hour

Additionally, using instance families optimized for specific workloads (e.g., C5 for compute-intensive tasks) can further reduce costs.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1

The company has a web application running on [[ec2]] instances behind an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]]. The infrastructure must scale horizontally during peak hours while minimizing costs during off-peak times. How would you address this challenge?

Correct Answer:

Implement an [[asg]] with [[ALB]] integration and a scaling policy based on [[cloudwatch|CloudWatch alarms]]. During off-peak hours, set minimum and maximum instance counts lower than during peak hours. This setup ensures efficient resource utilization and minimizes costs.

Incorrect Answers:

- Manually adding and removing instances based on demand: prone to human error and lacks automation benefits.
- Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]] without proper fallback mechanisms: could result in application downtime.

### Scenario 2

A media company needs to transcode videos from its users into multiple formats concurrently. They want to minimize the time required for transcoding while maintaining high reliability. What solution would you recommend?

Correct Answer:

Use [[aws-batch|AWS Batch]] with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|SPOT instances]] to perform video transcoding. By default, [[aws-batch|AWS Batch]] jobs run on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|SPOT instances]], which are significantly cheaper than On-Demand instances. If any [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|SPOT instances]] are terminated, [[aws-batch|AWS Batch]] automatically requests new ones.

Incorrect Answers:

- Creating an [[asg]] with manual scaling: not ideal for batch processing workloads.
- Implementing a [[lambda]] function for each video format conversion: might lead to cold start issues and limited concurrency.