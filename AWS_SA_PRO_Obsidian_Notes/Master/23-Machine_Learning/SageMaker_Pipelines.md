**Advanced Architecture: SageMaker Pipelines [[RDS_Instance_Types|Internals]]**

SageMaker Pipelines is a robust [[cicd|CI/CD]] service for machine learning (ML) workflows. It enables managing, deploying, and sharing end-to-end ML pipelines at any scale. Here's an in-depth look at its [[RDS_Instance_Types|internals]]:

* **Model Training & Deployment**: Based on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] training jobs and endpoints, it supports various algorithms, frameworks, and IPs.
* **Data Store**: Supports Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]], and [[redshift]] as data stores. Data can be preprocessed using built-in steps or custom scripts.
* **Version Control**: Git repositories ([[CodeCommit]], GitHub, etc.) store pipeline code, while Amazon ECR stores model images.
* **Orchestration**: Uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] under the hood, enabling long-running workflows, error handling, and parallel processing.
* **Integration**: Integrates with other services like [[cloudwatch]], [[AppConfig]], and ParallelCluster for monitoring, configuration management, and high-performance computing.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| SageMaker Pipelines | End-to-end ML workflow automation |
| Batch Transform | One-shot predictions without retraining |
| [[lambda]] + [[api-gateway|API Gateway]] | Real-time predictions |

Anti-patterns:

* Using SageMaker Pipelines for real-time predictions instead of Batch Transform or [[lambda]] + [[api-gateway|API Gateway]].
* Not considering throttling limits when designing pipelines.

**[[appsync|Security]] & Governance**

Implement granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and cross-account access:

```json
{
    "Effect": "Allow",
    "Action": [
      "sagemaker:*"
    ],
    "Resource": [
      "*"
    ],
    "Condition": {
        "StringEqualsIfExists": {
            "sagemaker:resource-type": [
                "Notebook",
                "TrainingJob",
                "TransformJob",
                "Endpoint",
                "EndpointConfig",
                "Model",
                "ModelPackage",
                "ModelPackageGroup"
            ]
        }
    }
}
```

Use Service Control [[policies]] (SCPs) in [[organizations|AWS Organizations]] to enforce [[control-tower|guardrails]] across accounts.

**Performance & Reliability**

To manage throttling limits, implement exponential backoff strategies when handling failures. For High Availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] ([[dr]]) patterns, distribute pipeline components across multiple regions and leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] for failover.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include:

* Enforcing resource quotas per user or team.
* Configuring lifecycle [[policies]] for resources such as endpoints, models, and notebooks.
* Monitoring costs using [[billing|Cost Explorer]] and Trusted Advisor.

Calculate costs by estimating the number of pipeline executions, storage requirements, and compute usage.

**Professional Exam Scenarios**

Scenario 1: A retail company wants to build a demand forecasting solution that automatically trains models based on new sales data daily. They want to ensure [[appsync|security]] and governance [[iam|best practices]] are applied.

Correct answer: Implement a SageMaker Pipeline with automated scheduling using [[cloudwatch]] Events. Apply [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions to secure resources. Use [[CodeCommit]] for version control and Amazon [[eventbridge]] to trigger the pipeline upon new dataset arrival.

Incorrect answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to train models based on new data since they don't support automated scheduling and have limited integration capabilities.

Scenario 2: A pharmaceutical firm needs to process sensitive genomic data for drug discovery. The data must remain within their [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and cannot leave the region.

Correct answer: Use SageMaker Processing Jobs and Notebooks within the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. Keep all data within the region and set up cross-account access if necessary.

Incorrect answer: Use SageMaker Pipelines since it uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] under the hood, which does not natively support [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] connectivity.