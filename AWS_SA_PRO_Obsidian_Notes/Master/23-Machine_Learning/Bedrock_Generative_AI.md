**Advanced Bedrock (Generative AI) Architecture**

Bedrock is a Generative AI service that allows developers to create sophisticated models for various use cases such as text generation, image synthesis, and recommendation systems. Understanding its [[RDS_Instance_Types|internals]] can help us build scalable and efficient solutions.

* Model Training*: Train custom models using your data or pre-built models from the model zoo. Fine-tune existing models for specific tasks.
* Data Parallelism*: Distribute training across multiple GPUs within a single instance for faster convergence.
* Synchronous/Asynchronous Training*: Choose between synchronous or asynchronous training depending on the time sensitivity of results and available resources.
* Distributed Training*: Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]]'s built-in support for distributed training, including HorovodRunner and TensorFlow FancyUV, to train large models over multiple instances.

**Comparison & Anti-Patterns**

| Service | Use Case |
|---|---|
| Bedrock (Generative AI) | High-quality text, image, and audio generation. Large-scale ML projects requiring multi-GPU training. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Rekognition]] | Image analysis, object detection, facial recognition. |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/16-other/textract|Amazon Textract]] | Document text extraction, form data capture. |

*Anti-pattern*: Using Bedrock for simple image processing tasks better suited for [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Rekognition|Rekognition]] or [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Textract|Textract]].

**[[appsync|Security]] & Governance**

Implement least privilege [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for users, groups, and roles. For example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "bedrock:CreateModel",
              "bedrock:StartTrainingJob"
            ],
            "Resource": [
              "*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": [
                "bedrock:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalTag/DataScientist": "true"
                }
            }
        }
    ]
}
```
Cross-account access requires creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the trusted account, allowing specified actions, and assuming the role from the source account.

Organization Service Control [[policies]] (SCPs) may be used to enforce [[control-tower|guardrails]] like restricting Bedrock usage to specific regions or limiting the number of concurrent training jobs.

**Performance & Reliability**

Bedrock supports throttling limits based on the type of operation. To handle throttled requests, implement exponential backoff strategies using AWS SDKs.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include running multiple Bedrock endpoints in different Availability Zones (AZs) and replicating training data across Regions.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls involve monitoring Bedrock usage through [[cloudwatch]] metrics, setting up [[billing]] alarms, and applying tags for cost allocation. Calculate costs by multiplying the hourly rate per instance by the duration in hours.

**Professional Exam Scenario #1**

Your company wants to deploy a generative AI model capable of generating high-quality images for their e-commerce platform. They expect millions of requests daily. Which architecture would you recommend?

Recommended Architecture:

* Deploy Bedrock endpoint(s) in auto-scaling groups spread across at least three AZs to ensure high availability and reliability.
* Implement multi-layer [[appsync|security]] with [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]], private subnets, and [[appsync|security]] groups.
* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]]'s distributed training capabilities to speed up model training.
* Apply granular cost controls using tags, [[Budgets]], and alerts.

Incorrect Answers:

* Deploying the model without proper scaling mechanisms will result in poor performance during peak times.
* Not distributing training over multiple GPUs could lead to longer training periods, increasing costs.

**Professional Exam Scenario #2**

Your organization needs to develop a generative AI model to generate synthetic medical images for research purposes. The team consists of data scientists, researchers, and clinicians. How would you secure the solution while maintaining ease of access for the team members?

Secure Solution:

* Create separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions for each user group (data scientists, researchers, clinicians).
* Implement SCPs to restrict Bedrock usage to specific regions and limit concurrent training jobs.
* Set up cross-account access for necessary data sharing.
* Utilize [[organizations|AWS Organizations]] to manage accounts and apply SCPs centrally.