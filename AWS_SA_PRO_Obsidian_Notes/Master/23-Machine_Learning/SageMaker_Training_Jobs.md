### Advanced Architecture

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] Training Job is a fully managed service that uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|SPOT instances]] or On-Demand instances to train machine learning models. The training jobs run on distributed or single-node hardware infrastructure. The [[RDS_Instance_Types|internals]] of SageMaker Training Job include Algorithm Kernel Image (AKI) which contains the algorithm logic, dependencies, and required libraries. The data preprocessing happens in the host container while the model training occurs within the AKI. The data can be stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and the results of the training job are saved back into [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

[[RDS_Instance_Types|Global scale considerations]] involve using Amazon [[fsx|FSx for Lustre]] as a high-performance file system for large datasets, which can be mounted across multiple Availability Zones (AZs) in a region. This ensures low-latency access to data during training. For cross-region data transfer, [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or [[Snowball]] devices can be used. To achieve global scale, you can set up separate training jobs in different regions, each utilizing local resources.

### Comparison & Anti-Patterns

| Feature | SageMaker Training Job | Alternatives |
| --- | --- | --- |
| Integration with [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] | Highly integrated with other services like [[Srinivas_Notes/S3|S3]], [[cloudwatch]], [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]], and Pipeline | Custom solutions using [[ec2]], [[eks]], or [[lambda]] require additional effort to integrate with these services |
| Scalability | Automatically scales up and down based on requirements | Custom solutions may require manual scaling or custom scripts |
| Cost | Can be expensive due to high resource utilization | Custom solutions allow granular control over resource usage and costs |

Anti-patterns include running long-lived training jobs on On-Demand instances instead of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]], not leveraging data distribution techniques such as [[Fargate]] or [[emr]], and failing to implement proper monitoring and alerting mechanisms.

### [[appsync|Security]] & Governance

To enforce [[appsync|security]] and governance [[iam|best practices]], create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] granting permissions to specific resources. Here's an example JSON policy snippet allowing SageMaker Training Job access to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:CreateTrainingJob",
                "sagemaker:DescribeTrainingJob"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "sagemaker:input-data-config.DataSource.S3DataSource.S3DataType": "S3Prefix",
                    "sagemaker:input-data-config.DataSource.S3DataSource.S3Uri": [
                        "arn:aws:s3:::your-bucket/*"
                    ]
                }
            }
        }
    ]
}
```
For cross-account access, create a role in the source account and attach a trust relationship policy allowing the destination account to perform actions. In the destination account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user or role with necessary permissions to assume the role in the source account.

Use Service Control [[policies]] (SCPs) at the organization level to restrict access to SageMaker Training Job or limit resource usage.

### Performance & Reliability

Throttling limits for SageMaker Training Job depend on the instance type and number of instances utilized. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies using AWS SDKs or third-party libraries.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve replicating SageMaker resources across multiple regions and using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and failover configurations.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include choosing the right instance types and sizes, using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] when possible, and setting up budget alerts. Calculate costs by analyzing the [[billing]] dashboard, focusing on charges related to SageMaker Training Job, including instance usage, storage, and data transfer fees.

### Professional Exam Scenario: Vignette-Style Questions

**Scenario 1:** Your company needs to train machine learning models using large datasets. Due to the size of the dataset, you need to distribute the data efficiently among multiple nodes. Which architecture should you choose?

*Correct answer:* Implement a distributed SageMaker Training Job with [[Fargate]] or [[emr]] as the data distribution mechanism. This allows efficient data processing and reduces overall training time.

*Incorrect answer:* Create a custom solution using [[ec2]] instances and manually distribute the data. This approach requires more effort and lacks the scalability benefits provided by SageMaker Training Job.

**Scenario 2:** Your organization has strict [[appsync|security]] [[policies]] requiring cross-account access to SageMaker Training Job resources. How would you implement this securely?

*Correct answer:* Set up an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with a trust relationship policy allowing the destination account to assume the role. Then, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user or role in the destination account with necessary permissions to assume the role in the source account.

*Incorrect answer:* Share the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket containing the training data directly with the destination account. While this method provides cross-account access, it does not follow the principle of least privilege and introduces unnecessary risks.