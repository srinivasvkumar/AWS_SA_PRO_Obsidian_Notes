**ECR Lifecycle [[policies]]: Advanced Architecture & [[RDS_Instance_Types|Internals]]**

At its core, ECR is a regional registry of container images per AWS account. An image is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software. Each image consists of one or more layers, and each layer is stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

Internally, ECR uses a combination of services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]], [[sts]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], [[cloudwatch]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and [[kms]] to provide a secure, scalable, and efficient experience. Images pushed to ECR are stored in an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket owned by AWS but accessible only from ECR APIs. The ECR metadata, such as image manifest and tagging information, are stored in [[dynamodb|DynamoDB tables]].

ECR replication allows you to automatically replicate images across regions. This feature creates a logical "ECR repository" that spans multiple physical ECR registries in different regions. Replication can be configured using lifecycle [[policies]].

**ECR Lifecycle [[policies]]: Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[ecs]] | If your application runs on [[ec2]] instances and requires fine-grained control over the host OS, network configuration, etc. |
| [[Fargate]] | If you want to focus on writing applications without worrying about servers, scaling, patching, etc. |
| [[lambda]] | If your application has an event-driven serverless architecture. |
| Batch | If you need to perform batch computing workloads. |

Anti-patterns include running stateful services inside containers, storing large amounts of data within containers, and managing container configurations manually instead of automating them.

**ECR Lifecycle [[policies]]: [[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting permissions to specific users, groups, or roles at various levels. Here's an example policy allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to manage ECR repositories:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetLifecyclePolicy",
                "ecr:GetLifecyclePolicyPreview",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:DescribeRepositories",
                "ecr:BatchGetImage",
                "ecr:CreateRepository",
                "ecr:DeleteRepository",
                "ecr:DescribeRegistry",
                "ecr:SetRepositoryPolicy",
                "ecr:DeleteRepositoryPolicy",
                "ecr:UpdateLifecyclePolicy",
                "ecr:PutImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access involves creating a resource policy on the source repository to allow specified principals from other accounts to interact with it. For instance, here's how to add a principal from another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPrincipal",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage"
            ],
            "Resource": "arn:aws:ecr:us-west-2:123456789012:repository/my-repository",
            "Condition": null
        }
    ]
}
```

Organization Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] around ECR usage. For example, preventing the creation of public repositories:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PreventPublicECRRepositories",
            "Effect": "Deny",
            "Action": [
                "ecr:CreateRepository"
            ],
            "Resource": [
                "arn:aws:ecr:*::repository/*"
            ],
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "aws:RequestedRegion": [
                        "*"
                    ]
                },
                "Bool": {
                    "aws:ViaAWSService": [
                        "true"
                    ]
                },
                "Null": {
                    "aws:SourceVpce": []
                }
            }
        }
    ]
}
```

**ECR Lifecycle [[policies]]: Performance & Reliability**

Throttling limits in ECR depend on the operation. Most API calls have a rate limit of 10 requests per second, which can be increased through support requests. To handle throttling exceptions, implement exponential backoff strategies in your code.

HA/DR patterns involve using ECR replication to distribute images across multiple regions and availability zones. By default, all repositories have cross-region replication enabled. However, you can configure lifecycle [[policies]] to manage replication rules better.

**ECR Lifecycle [[policies]]: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting up [[billing]] alarms based on ECR usage and monitoring costs at the repository level. Additionally, cleaning up unused images and tags reduces storage costs.

Calculation examples:
- Storage costs: Number of repositories * Total storage size * Region storage price
- Data transfer costs: Number of image pushes * Push data size * Price per GB

**Professional Exam Scenario 1**

Suppose you have two AWS accounts, A and B, where Account A contains a shared container library used by Account B. How would you set up cross-account access?

*Correct answer*: Create a resource policy on the shared repository in Account A to allow specified principals from Account B. Then, attach the necessary permissions to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role or user in Account B.

Incorrect answers might include using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]], sharing via [[organizations|AWS Organizations]], or configuring [[sts]] roles.

**Professional Exam Scenario 2**

As part of a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] strategy, you must ensure that all container images in Account A are available in Account B. What steps should you take?

*Correct answer*: Enable ECR replication between Account A and Account B, then create a lifecycle policy in Account A to replicate images to Account B.

Incorrect answers might include using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 replication]], manually copying images, or setting up [[ecs]] replication.