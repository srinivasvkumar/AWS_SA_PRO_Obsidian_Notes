**Advanced Architecture**

[[emr]] Serverless is a fully managed service that simplifies running Spark applications without managing servers or clusters. It uses Amazon [[emr]]'s underlying technology but abstracts away infrastructure management. The architecture involves auto-scaling of compute resources based on workload requirements.

Internally, it uses a multi-tenant architecture with shared resources like storage, reducing costs compared to traditional [[emr]] clusters. Data processing happens in containers managed by Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] (Elastic File System) for shared file system access across containers.

For [[RDS_Instance_Types|global scale considerations]], [[emr]] Serverless leverages AWS's global network infrastructure through regional endpoints. However, data locality isn't guaranteed due to its multi-tenant nature. Therefore, it's not recommended for applications needing strict data residency compliance.

**Comparison & Anti-Patterns**

| Service | Use Case |
|---|---|
| [[emr]] Serverless | Ad-hoc data processing, real-time analytics, machine learning tasks, and ephemeral workloads. |
| Glue/Athena | Data cataloging, SQL querying over [[Srinivas_Notes/S3|S3]], serverless ETL workflows. |
| [[kinesis|Kinesis Data Analytics]] | Real-time stream processing. |

Anti-patterns include long-running jobs, high-throughput steady state workloads, and applications requiring dedicated hardware resources.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using [[cloudformation|conditions]] and nested policy elements. Here's an example denying specific actions within a bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::example-bucket/*"
            ],
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "aws:SourceVpce": "vpce-1234567890abcdef0"
                }
            }
        }
    ]
}
```

Cross-account access should be granted via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] around usage.

**Performance & Reliability**

Throttling limits exist per account and region. If exceeded, requests will fail. Implement exponential backoff strategies in your application code to handle these failures gracefully.

HA/DR patterns involve deploying applications across multiple availability zones or regions. For [[emr]] Serverless, since storage is decoupled from compute, you could replicate data to another region and run new jobs there.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by setting up alerts on AWS [[billing|Cost Explorer]] when usage surpasses predefined thresholds. Also, you can calculate costs using the following formula:

Cost = (Duration / 60 minutes) * NumberOfTasks * TaskMemory * TaskVCPU * PricePerGBSecond

**Professional Exam Scenarios**

Scenario 1: A data engineering team wants to process large datasets periodically while minimizing cost. They currently use [[emr]] clusters but find them expensive and hard to manage. How would you help them transition to [[emr]] Serverless?

Correct Answer: Transitioning to [[emr]] Serverless reduces operational overhead and scales automatically based on workload demands. This results in lower costs compared to provisioned [[emr]] clusters.

Incorrect Answer: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] instead of [[emr]] Serverless might reduce costs but doesn't eliminate cluster management complexity.

Scenario 2: An organization has strict data residency requirements and uses [[emr]] Serverless for ad-hoc data processing. How can they ensure their data remains within their home region?

Correct Answer: Since [[emr]] Serverless doesn't guarantee data locality, it's not suitable for applications needing strict data residency compliance. Instead, they should consider alternative services like [[emr]] on [[ec2]] with instances tied to specific availability zones.

Incorrect Answer: Configuring [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] for [[emr]] Serverless doesn't restrict data movement within a region as [[emr]] Serverless doesn't support [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]].