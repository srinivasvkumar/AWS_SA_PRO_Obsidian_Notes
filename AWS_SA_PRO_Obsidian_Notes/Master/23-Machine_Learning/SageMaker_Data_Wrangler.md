**[[RDS_Instance_Types|1. Advanced Architecture]]: SageMaker Data Wrangler**

SageMaker Data Wrangler is a fully managed data preprocessing tool that allows data scientists and engineers to clean, transform, and prepare data for machine learning models. It provides a visual interface to apply various preprocessing techniques and generates optimized code in Python or Spark, which can be further customized.

Internally, Data Wrangler leverages [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] Processing Jobs under the hood. Each processing job runs an instance of a Docker container defined by a `processing` job definition. This definition specifies the image, role, and resources required to execute the job. The input data is stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and the output is written back to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] upon completion.

For [[RDS_Instance_Types|global scale considerations]], Data Wrangler supports using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share Data Wrangler flows across accounts within an organization. This enables central management of common data [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|preparation]] tasks while allowing individual teams to maintain their own SageMaker resources.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]: SageMaker Data Wrangler vs Alternatives**

| Service | Use Cases | Anti-Patterns |
| --- | --- | --- |
| SageMaker Data Wrangler | Simple to complex data preprocessing tasks requiring visual tools. Integrates well with other SageMaker services. | Real-time data preprocessing, large-scale ETL workloads, non-machine learning specific tasks. |
| [[glue|AWS Glue]] | Large-scale ETL workloads, real-time data processing, serverless architecture. | Requires programming skills, not designed for machine learning specific tasks. |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/08-data-analytics/others|AWS Data Pipeline]] | Orchestration of data movement between different AWS compute and storage services. | Not suitable for machine learning specific tasks. |

**[[RDS_Instance_Types|3. Security & Governance]]: SageMaker Data Wrangler**

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Data Wrangler, you can create a custom SageMaker execution role with permissions scoped to specific [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets and necessary AWS services. Here's an example JSON policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:CreateProcessingJob",
                "sagemaker:DescribeProcessingJob"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::example-input-bucket/*",
                "arn:aws:s3:::example-output-bucket"
            ]
        }
    ]
}
```

Cross-account access to shared Data Wrangler flows can be achieved through AWS [[ram]] by creating a resource share and adding the desired accounts as members. To enforce [[appsync|security]] and governance at scale, use Organization Service Control [[policies]] (SCPs) to restrict the creation of SageMaker resources outside approved accounts or regions.

**[[RDS_Instance_Types|4. Performance & Reliability]]: SageMaker Data Wrangler**

Data Wrangler has throttling limits based on the number of concurrent processing jobs. These limits depend on the SageMaker service quota assigned to your account. To manage performance during high load periods, implement exponential backoff strategies when encountering throttled requests.

HA/DR patterns for SageMaker Data Wrangler involve using multiple AWS Regions and Accounts for redundancy and separation of concerns. For example, store input data in a central [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket replicated across multiple regions and use cross-region replication for backup purposes.

**[[RDS_Instance_Types|5. Cost Optimization]]: SageMaker Data Wrangler**

Granular cost controls for SageMaker Data Wrangler include setting up [[billing]] alarms, monitoring usage trends, and applying tags to track costs associated with specific projects or teams. Additionally, you can take advantage of SageMaker [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for processing jobs to reduce costs significantly.

**6. Professional Exam Scenario: SageMaker Data Wrangler**

Scenario A: A data scientist wants to use SageMaker Data Wrangler but requires access to a shared flow created by another team. How would you set up cross-account access?

Correct Answer: Use AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share the Data Wrangler flow from the source account with the destination account. In the destination account, create a resource import request to import the shared flow.

Incorrect Answer: Modify the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in both accounts to grant permissions to each other's resources. This approach is incorrect because it introduces unnecessary trust relationships between the two accounts.

Scenario B: A company needs to process large datasets in SageMaker Data Wrangler while minimizing costs. What [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] strategy would you recommend?

Correct Answer: Utilize SageMaker [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] for processing jobs when possible. Set up [[billing]] alarms and monitor usage trends to ensure cost control. Implement tagging strategies to track costs associated with specific projects or teams.

Incorrect Answer: Store all datasets in a single [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and enable versioning to minimize costs. This answer is incorrect because storing data in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] does not directly contribute to reducing Data Wrangler processing costs.