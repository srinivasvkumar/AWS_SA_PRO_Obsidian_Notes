## Advanced Architecture
At its core, ECR Image Scanning is a container image vulnerability scanning service provided by AWS. It uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Inspector]]'s scanning capabilities to analyze container images in your ECR repositories for potential [[appsync|security]] risks. The service checks for any known [[appsync|security]] issues in the base image or any installed packages within the container.

The [[RDS_Instance_Types|internals]] of ECR Image Scanning involve an event-based system that triggers image scans when certain events occur. These events can be uploading a new image to an ECR repository, or it can be scheduled at regular intervals. The actual scanning process involves comparing the contents of the container image against a database of known vulnerabilities, such as the National Vulnerability Database (NVD) maintained by NIST.

[[RDS_Instance_Types|Global scale considerations]] are handled automatically by AWS. When you use ECR Image Scanning in a multi-region environment, each region operates independently with its own ECR service and associated resources. This means that if you have ECR repositories in multiple regions, you will need to enable image scanning separately for each one. However, the underlying vulnerability databases used for scanning are centrally managed by AWS, ensuring consistent results across all regions.

## Comparison & Anti-Patterns
Here are some comparison points between ECR Image Scanning and alternative services:

| Service | Use Case |
|---|---|
| ECR Image Scanning | Best for users who want to store and manage their container images within the AWS ecosystem. |
| Docker Hub Container Scanning | Suitable for users who primarily use Docker Hub for container image management. |
| GitHub Dependabot | Ideal for open source projects hosted on GitHub that require dependency updates. |

Anti-patterns include using ECR Image Scanning as a standalone [[appsync|security]] solution without implementing other [[appsync|security]] [[iam|best practices]] like principle of least privilege, network segmentation, and runtime [[appsync|security]] enforcement.

## [[appsync|Security]] & Governance
[[appsync|Security]] and governance with ECR Image Scanning involves setting up complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and organization service control [[policies]] (SCPs). Here's an example JSON policy snippet that restricts image scanning to specific users:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeImages",
                "ecr:DescribeRepositories",
                "ecr:GetDownloadUrlForLayer",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEqualsIfExists": {
                    "aws:SourceVaultARN": [
                        "arn:aws:s3:::your-bucket-name"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecr:StartImageScan",
                "ecr:BatchGetImage",
                "ecr:DescribeImageScanFindings"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": [
                        "arn:aws:ecr:us-west-2:123456789012:repository/*"
                    ]
                }
            }
        }
    ]
}
```
Cross-account access can be enabled by adding appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions in both accounts. Organization SCPs can be used to enforce image scanning [[policies]] across all member accounts.

## Performance & Reliability
Throttling limits in ECR Image Scanning depend on the number of images being scanned and the frequency of scanning. If you exceed these limits, you may experience throttling [[api-gateway|errors]]. To mitigate these issues, implement exponential backoff strategies to retry failed requests.

HA/DR patterns for ECR Image Scanning involve storing critical container images in multiple regions and enabling image scanning in each region. This ensures high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls in ECR Image Scanning involve monitoring usage patterns and optimizing image scanning based on those patterns. For example, if you only upload new images once a week, you could schedule image scanning to occur once a week instead of continuously. Calculating costs involves understanding the pricing model for ECR Image Scanning, which includes charges for data storage and data transfer.

## Professional Exam Scenarios
### Scenario 1
You are working as a solutions architect for a large financial institution that has strict compliance requirements. They want to scan their container images for vulnerabilities but don't want to store them outside of their own [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. What solution would you recommend?

Correct Answer: Use ECR Image Scanning in combination with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint for ECR. This allows the financial institution to keep their container images within their own [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] while still taking advantage of ECR Image Scanning's vulnerability scanning capabilities.

Incorrect Answer: Use Docker Hub Container Scanning. While this solution provides container scanning capabilities, it does not meet the requirement of keeping container images within the financial institution's own [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

### Scenario 2
Your company is a small startup that relies heavily on open source software. They want to ensure that their container images do not contain any known vulnerabilities before deploying them to production. What solution would you recommend?

Correct Answer: Use ECR Image Scanning in combination with a continuous integration/continuous deployment ([[cicd|CI/CD]]) pipeline. This allows the startup to automatically scan their container images during the build process and fail the build if any vulnerabilities are detected.

Incorrect Answer: Use GitHub Dependabot. While this solution provides dependency updates for open source projects, it does not provide comprehensive container image scanning capabilities.