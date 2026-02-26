**[[RDS_Instance_Types|1. Advanced Architecture]]**

SageMaker Notebooks is a fully managed service that provides Jupyter notebooks for data science and machine learning (ML) tasks. It supports multiple users and enables them to share their findings, making it an excellent choice for team-based projects. The architecture consists of several components, including ML compute instances, Amazon Elastic File System ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]), and Amazon [[fsx|FSx for Lustre]] file systems.

At the core of SageMaker Notebooks is the Notebook Instance, which includes a ML compute instance and a persistent storage volume. Each Notebook Instance has an associated lifecycle configuration that specifies scripts to run when the Notebook Instance is created or started. This allows you to install additional software packages, manage dependencies, and perform other setup tasks.

[[RDS_Instance_Types|Global Scale Considerations]]:

* Multi-region and multi-account deployment using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share resources across accounts
* Using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] to transfer data between regions or accounts
* Integration with [[glue|AWS Glue]] to create ETL jobs and manage metadata

Under the Hood Mechanics:

* Uses Amazon Linux AMI optimized for ML workloads
* Automatically provisions and manages infrastructure required for running Jupyter Notebook servers
* Supports various ML frameworks such as TensorFlow, PyTorch, and Apache MXNet

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
|---|---|
| SageMaker Notebooks | Collaborative ML development, experimentation, and prototyping |
| [[ec2]] for Deep Learning (DL1) | DL training at scale with high memory requirements |
| [[lambda|AWS Lambda]] | Running small ML inference tasks without managing infrastructure |

Anti-Patterns:

* Sharing Notebook Instances via email or copy-paste instead of using SageMaker's built-in sharing capabilities
* Storing sensitive information directly in the Notebook Instance instead of using [[secrets-manager|AWS Secrets Manager]] or [[parameter-store|AWS Systems Manager Parameter Store]]

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
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
        }
    ]
}
```
Cross-Account Access:

* Create a role in the source account with necessary permissions and attach a trust policy allowing the destination account to assume the role
* In the destination account, create a role with permissions to access the shared resources and add an inline policy granting permission to assume the role from the source account

Organization SCPs:

* Implement centralized control over resource creation by configuring SCPs in the organization root or OU level
* Prevent unauthorized access to resources by restricting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity actions and resource ARN usage

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:

* Monitor throttling exceptions using [[cloudwatch|CloudWatch Alarms]] and implement exponential backoff strategies to handle these situations

HA/DR Patterns:

* Enable termination protection for Notebook Instances to prevent accidental deletion
* Back up critical data using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots and store them in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for long-term retention
* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] to automate backup schedules and recovery processes

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular Cost Controls:

* Set up [[billing]] alerts to track monthly costs
* Choose appropriate instance types based on your workload requirements
* Schedule Notebook Instances to start and stop automatically based on usage patterns

Calculation Examples:

* Estimate hourly instance costs: `(instance_type_cost * number_of_instances * hours_used) + (volume_size_gb * volume_type_cost)`
* Calculate total monthly cost: `(hourly_cost * days_in_month * 24) + (data_transfer_cost * GB_transferred)`

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: Suppose your organization operates in two separate AWS accounts, one for production and another for development. You want to enable collaboration between teams working on ML projects while maintaining [[appsync|security]] [[iam|best practices]]. How would you accomplish this?

Correct Answer: Use AWS Resource Access Manager to share SageMaker Notebook resources between the two accounts. Additionally, configure cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]].

Incorrect Answer: Share Notebook Instances via email or copy-paste.

Scenario 2: Your company has a large dataset stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] that needs to be processed by a SageMaker Notebook. However, the dataset size exceeds the available storage space in the SageMaker Notebook's volume. How can you efficiently process this dataset within the Notebook?

Correct Answer: Use Amazon [[fsx|FSx for Lustre]] or Amazon Elastic File System ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]) to extend the storage capacity of the SageMaker Notebook.

Incorrect Answer: Modify the SageMaker Notebook's instance type to increase its storage capacity.