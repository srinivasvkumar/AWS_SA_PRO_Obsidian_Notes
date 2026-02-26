## Advanced Architecture

At its core, Amazon Elastic Container Registry (ECR) Public is a managed container image storage and distribution service that allows developers to host, share, and deploy container images across various environments. ECR Public operates as a distinct service from ECR, targeting scenarios involving public container repositories and enabling sharing of these resources with other AWS accounts or anonymous users.

Internally, ECR Public utilizes Amazon Simple Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]) to store container images in a highly durable and scalable manner. The ECR Public Registry acts as an abstraction layer over [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], offering features like automatic Docker image generation, encryption, and lifecycle management. Furthermore, ECR Public employs a global content delivery network (CDN) powered by Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to ensure efficient and performant retrieval of container images worldwide.

To achieve global scale, ECR Public Registry uses multiple regions and availability zones (AZs) for storing and distributing container images. This setup enables low-latency access to container images while maintaining high levels of durability and availability.

## Comparison & Anti-Patterns

Here's a comparison table highlighting when not to use ECR Public Registry and possible alternatives:

| Use Case                          | ECR Public Registry                | Alternatives                         |
| ---------------------------------- | ---------------------------------- | ------------------------------------ |
| Private container repository       | ECR (not ECR Public Registry)      | GitHub Packages, Google Container Registry   |
| Sharing between AWS accounts        | ECR Public Registry                | ECR Resource [[policies]]                |
| Anonymous user access              | ECR Public Registry                | [[Artifact]] Hub, Docker Hub             |
| Multi-platform compatibility       | Docker Hub, Google Container Registry | ECR Public Registry (with conversion tools) |

Common anti-patterns include using ECR Public Registry for private container repositories and employing it as a default option without considering alternative solutions.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created to manage access to ECR Public Registry resources. For example, you might grant anonymous users read-only access to specific repositories using the following JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "ecr-public:BatchCheckLayerAvailability",
                "ecr-public:GetDownloadUrlForLayer"
            ],
            "Resource": "arn:aws:ecr-public:us-west-2:123456789012:repository/my-repo/*"
        }
    ]
}
```

Cross-account access can also be established using resource-based [[policies]] attached to individual repositories. For instance, you could allow another account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to pull images from your ECR Public Registry repository:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111122223333:user/other-user"
            },
            "Action": [
                "ecr-public:BatchCheckLayerAvailability",
                "ecr-public:GetDownloadUrlForLayer"
            ],
            "Resource": "arn:aws:ecr-public:us-west-2:123456789012:repository/my-repo/*"
        }
    ]
}
```

Lastly, organization Service Control [[policies]] (SCPs) may be used to enforce granular control over ECR Public Registry usage within an [[AWS Organization]].

## Performance & Reliability

ECR Public Registry enforces throttling limits to prevent overwhelming the underlying infrastructure. To avoid exceeding these limits, implement exponential backoff strategies when making requests to ECR Public Registry APIs. Additionally, ensure proper [[api-gateway|caching]] of container images at the CDN edge locations and leverage Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] for high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] (HA/DR) purposes.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved through monitoring and analyzing API calls, data transfer costs, and storage consumption. Here's a calculation example based on the AWS pricing page:

* API Calls: $0.10 per 1,000 calls
* Data Transfer (US West): $0.09 per GB
* Storage (per GB per month): $0.01

Suppose you maintain 100 container images, each with a size of 500 MB, and make 100,000 API calls per month. Your monthly cost estimate would look like this:

* API Calls: 100,000 \* $0.10 / 1,000 = $10.00
* Data Transfer (assuming 10 GB downloaded): 10 \* $0.09 = $0.90
* Storage (50 GB total): 50 \* $0.01 = $0.50

Total estimated monthly cost: $11.40

## Professional Exam Scenarios

### Scenario 1

Your company has recently open-sourced one of its software components and wants to distribute it via a public container registry. They have asked you to recommend a solution that meets their requirements:

* Global availability
* High reliability
* Low latency
* Integration with [[cicd|CI/CD]] pipelines

#### Solution

Use ECR Public Registry to host the container images, allowing global availability through Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] integration. Implement high reliability by leveraging Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]], and reduce latency by [[api-gateway|caching]] container images at CDN edge locations. Lastly, integrate ECR Public Registry with [[cicd|CI/CD]] pipelines to automate the build, test, and deployment processes.

#### Incorrect Answer Justification

Using ECR (non-Public Registry) does not meet the requirement for public access. Using self-hosted options such as Docker Hub or GitHub Packages may result in higher maintenance overhead and lack customizability compared to ECR Public Registry.

### Scenario 2

Your organization manages multiple AWS accounts and requires a centralized container image repository accessible across all accounts. They want to minimize cross-account access configuration complexity.

#### Solution

Create a single ECR Public Registry repository and apply resource-based permissions to enable cross-account access. Attach a resource policy to the repository that grants the necessary permissions to relevant [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in different accounts.

#### Incorrect Answer Justification

Implementing separate ECR Public Registry repositories per account increases operational complexity and maintenance efforts. Utilizing private ECR repositories instead of ECR Public Registry requires additional manual effort to configure cross-account access.