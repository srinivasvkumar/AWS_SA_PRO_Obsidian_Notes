**Advanced Architecture: SageMaker Edge Manager**

SageMaker Edge Manager is a service that allows you to manage ML models deployed on edge devices. The [[RDS_Instance_Types|internals]] of SageMaker Edge Manager include a cloud-based console, MLOps service, and edge agent. The console provides a UI to register, deploy, and monitor models. The MLOps service enables model packaging, validation, and deployment. The edge agent is installed on edge devices and communicates with the MLOps service for model updates and monitoring.

[[RDS_Instance_Types|Global scale considerations]] involve deploying the MLOps service in a multi-region architecture. This requires configuring endpoints in each region and managing them using [[organizations|AWS Organizations]] or Service Control [[policies]] (SCPs). Under the hood, SageMager Edge Manager uses [[iot|AWS Greengrass]] for device management and [[iot|AWS IoT Core]] for connectivity.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| SageMaker Edge Manager | Managing ML models on edge devices at scale |
| [[lambda|AWS Lambda]] | Serverless compute for lightweight workloads |
| Amazon [[ecs]] | Container orchestration for custom workflows |

Anti-patterns include using SageMaker Edge Manager for non-ML workloads, not considering throttling limits, and failing to implement exponential backoff strategies for device connectivity issues.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing cross-account access using AssumeRole and [[sts]] Assumerolewithwebidentity. JSON snippet:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "prod-access"
                }
            }
        }
    ]
}
```
Cross-account access can be managed using [[organizations|AWS Organizations]] and SCPs. For example, an [[SCP]] can be used to enforce usage of specific SageMaker features.

**Performance & Reliability**

Throttling limits for SageMaker Edge Manager include a maximum number of edge agents per account and concurrent API calls. Exponential backoff strategies should be implemented when encountering throttling [[api-gateway|errors]]. High availability/disaster recovery patterns involve deploying edge agents across multiple regions and using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] for data protection.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting up [[billing]] alerts and using AWS [[billing|Cost Explorer]] to analyze usage trends. Calculation examples:

* Edge agent cost = number of edge agents \* cost per edge agent
* Data transfer cost = amount of data transferred \* cost per GB

**Professional Exam Scenarios**

Scenario 1: A company wants to deploy ML models on edge devices for real-time predictions. They need to manage these models at scale and monitor their performance. Which service should they use? Why?

Correct answer: SageMaker Edge Manager. It allows for managing ML models on edge devices at scale and includes a cloud-based console for registration, deployment, and monitoring.

Incorrect answer: [[lambda|AWS Lambda]]. It is designed for serverless compute for lightweight workloads and does not support ML model management on edge devices.

Scenario 2: A company has deployed SageMaker Edge Manager in one region and wants to replicate the setup in another region. How can they achieve this while ensuring [[appsync|security]] and governance?

Correct answer: Use [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to enforce usage of specific SageMaker features in both regions. Additionally, configure endpoints in each region and manage them using the SageMaker Edge Manager console.

Incorrect answer: Use [[AWS Systems Manager]] to manage the setup in both regions. While Systems Manager can be used for patching and configuration, it does not provide the same level of ML model management capabilities as SageMaker Edge Manager.