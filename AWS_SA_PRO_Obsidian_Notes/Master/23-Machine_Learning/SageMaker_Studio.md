**Advanced Architecture**

SageMaker Studio is a web-based, integrated development environment (IDE) for machine learning (ML) that enables users to build, train, and deploy ML models at scale. It provides a unified environment for all ML tasks, allowing data scientists, developers, and engineers to perform end-to-end ML workflows without leaving the platform.

Internally, SageMaker Studio uses Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] clusters to manage resources for each user. Each user has their own [[ecs]] cluster, which can be scaled up or down based on the user's needs. The [[ecs]] cluster runs Docker containers for each user's compute instances, providing resource isolation and efficient sharing of resources.

At the global scale, SageMaker Studio supports multiple regions and accounts. To enable cross-region and cross-account collaboration, users can create domain configurations in one account and share them with other accounts using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]]. This allows teams to collaborate across different regions and accounts while maintaining separate [[billing]] and [[appsync|security]] boundaries.

To optimize performance and reliability, SageMaker Studio uses Amazon [[cloudwatch]] for monitoring and alerting. Users can set up alarms and notifications for various metrics such as CPU utilization, memory usage, and network traffic. In addition, SageMaker Studio automatically scales compute resources based on workload demand, ensuring optimal performance and minimizing costs.

**Comparison & Anti-Patterns**

Here are some scenarios when NOT to use SageMaker Studio and alternative services to consider:

| Scenario | Alternative Services |
| --- | --- |
| Small-scale ML projects | Local development environments, Jupyter Notebooks |
| Real-time inference | [[lambda|AWS Lambda]], [[api-gateway|API Gateway]] |
| Deep learning frameworks not supported by SageMaker | AWS [[ec2]] instances with custom AMIs |
| Batch processing | [[aws-batch|AWS Batch]], [[glue|AWS Glue]] |
| Data processing and transformation | [[glue|AWS Glue]], AWS DataBrew |

Common design mistakes include:

* Running compute instances in a single region instead of distributing them across multiple regions for high availability.
* Sharing SageMaker Studio resources with unauthorized users or groups.
* Using default settings for compute instances, resulting in overprovisioned resources and higher costs.

**[[appsync|Security]] & Governance**

To ensure secure access to SageMaker Studio, users can implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON snippets. For example, to allow a user to create and delete notebooks but not modify them, the following policy can be used:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:CreateNotebookInstance",
                "sagemaker:DeleteNotebookInstance"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": [
                "sagemaker:DescribeNotebookInstance",
                "sagemaker:UpdateNotebookInstance",
                "sagemaker:DeleteNotebookInstance"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEqualsIfExists": {
                    "sagemaker:NotebookInstanceName": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```
For cross-account access, users can use AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share domain configurations with other accounts. Additionally, they can use [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to enforce [[appsync|security]] [[policies]] across multiple accounts.

**Performance & Reliability**

To avoid throttling limits and ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], users can implement exponential backoff strategies when making requests to SageMaker Studio APIs. They can also distribute compute instances across multiple regions and use Amazon [[cloudwatch]] for monitoring and alerting.

In addition, users can implement high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns by replicating SageMaker Studio resources across multiple regions. For example, they can create a primary SageMaker Studio domain in one region and a secondary domain in another region. They can then synchronize resources between the two domains using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or custom scripts.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

To optimize costs, users can implement granular cost controls using AWS [[billing|Cost Explorer]] and [[billing|AWS Budgets]]. They can monitor usage trends, identify underutilized resources, and set [[Budgets]] and alerts for specific SageMaker Studio resources.

Here are some calculation examples:

* Compute instance costs: $0.90 per hour \* number of hours \* number of instances
* Data storage costs: $0.023 per GB per month \* number of GBs
* Training job costs: $0.000024 per GPU hour \* number of GPUs \* number of hours

**Professional Exam Scenarios**

Scenario 1: A biotech company wants to use SageMaker Studio to train machine learning models on genomic data. They need to ensure that the data is encrypted both at rest and in transit. What are the necessary steps to achieve this?

Correct answer:

* Enable encryption at rest for the SageMaker Studio domain and file system.
* Configure the SageMaker Studio notebook to use HTTPS for all communication.
* Encrypt the genomic data before uploading it to SageMaker Studio.

Incorrect answer:

* Disable encryption at rest for the SageMaker Studio domain and file system.
* Use HTTP for communication between the SageMaker Studio notebook and the SageMaker endpoint.
* Do not encrypt the genomic data before uploading it to SageMaker Studio.

Scenario 2: A retail company wants to use SageMaker Studio to train machine learning models on customer data distributed across multiple regions. They want to minimize latency and ensure high availability. What are the necessary steps to achieve this?

Correct answer:

* Distribute SageMaker Studio resources across multiple regions.
* Implement Amazon [[cloudwatch]] for monitoring and alerting.
* Implement exponential backoff strategies for SageMaker Studio API requests.

Incorrect answer:

* Keep all SageMaker Studio resources in a single region.
* Ignore monitoring and alerting.
* Do not implement exponential backoff strategies for SageMaker Studio API requests.