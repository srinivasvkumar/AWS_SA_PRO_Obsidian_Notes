**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[eks]] clusters are built using the Amazon [[eks]] Service and the Kubernetes control plane. The control plane consists of API servers, etcd datastore, and controllers that manage the worker nodes in your cluster. These components run across multiple AZs within an EKS-supported region.

Internally, [[eks]] uses core AWS services like [[ec2]], [[ALB]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] to provide functionality. For instance, [[eks]] creates ENI resources for pod networking, and [[appsync|Security]] Groups are used for controlling network ingress/egress at the node level.

When designing large-scale systems, there are several options available to distribute workloads across multiple regions or accounts:

- **Regional Multi-Cluster Strategy**: Deploy separate [[eks]] clusters per region for lower latency and higher availability. Data replication can be achieved through tools like Velero, backed by [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].
- **Account Multi-Cluster Strategy**: Create [[eks]] clusters in different AWS accounts for logical separation and resource management. This strategy requires cross-account access configuration.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Factor | [[ecs]] | [[eks]] | [[Fargate]] |
|---|---|---|---|
| Worker Nodes | Required | Not required | Not required |
| Container Orchestration | Built-in | Managed K8s | Managed K8s |
| Scaling Mechanism | [[ec2]] instances/ASGs | Node groups/Managed node groups | Per-container |
| Networking | [[Srinivas_Notes/VPC|VPC]], ENIs, [[appsync|Security]] Groups | [[Srinivas_Notes/VPC|VPC]], ENIs, CNI, [[appsync|Security]] Groups | [[Srinivas_Notes/VPC|VPC]], ENIs, [[appsync|Security]] Groups |

Anti-patterns include:

- Running stateful applications in stateless-oriented containers.
- Using [[eks]] when the simplicity of [[ecs]] is sufficient for the use case.
- Designing monolithic applications as containerized microservices.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow the principle of least privilege. Here's an example policy allowing [[eks]] access:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "eks:Describe*",
              "eks:List*"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access can be granted via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships between accounts. Additionally, you can enforce [[appsync|security]] standards using Service Control [[policies]] (SCPs) within [[organizations]].

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[eks]] provides throttling limits on various API calls. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies with jitter to avoid overloading APIs and ensure reliability.

HA/DR patterns involve deploying [[eks]] clusters in multiple regions and using [[route53]] [[route53|health checks]] for automatic failover. Alternatively, you may opt for manual failover by managing DNS records outside of AWS.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include:

- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for worker nodes: Up to 90% savings compared to On-Demand pricing.
- Auto Scaling: Adjust worker nodes based on utilization.
- Savings Plans: Commitment-based discounts for consistent workloads.

Example calculation for a single node group with 10 t3.medium workers:

- On-Demand: $0.0464/hour * 10 = $0.464/hour
- Spot (~50% discount): $0.0464/hour * 10 * 0.5 = $0.232/hour

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You need to deploy an application with strict [[appsync|security]] requirements while minimizing costs. The solution must support auto scaling and horizontal scaling. Which architecture would best meet these needs?

Correct Answer: [[eks]] with managed node groups utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]].
Justification: Offers Kubernetes-based container orchestration with granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] and autoscaling.

Incorrect Answers:

- [[ecs]]: Lacks native Kubernetes integration.
- [[Fargate]]: Does not offer direct cost savings from [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]].

Scenario 2:
Your company has multiple departments, each requiring isolated environments for their applications. How can you achieve account-level separation while maintaining centralized monitoring and management capabilities?

Correct Answer: Implement separate [[eks]] clusters in individual AWS accounts and configure cross-account access.
Justification: Provides logical separation and resource management while enabling centralized monitoring through [[cloudwatch]] and other third-party tools.

Incorrect Answers:

- Single [[eks]] cluster with namespaces: Shares underlying infrastructure, which could compromise isolation.
- Centralized ECR repository: Only solves part of the problem, leaving [[eks]] cluster deployment unaddressed.