## Advanced Architecture

At its core, Amazon Elastic Container Registry (ECR) is a fully-managed Docker container registry that enables you to store, retrieve, and deploy container images. ECR is integrated with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]], such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]], Batch, [[Fargate]], and [[lambda]], making it easy to utilize containerized applications across your environment.

### [[RDS_Instance_Types|Internals]]

ECR operates using two types of repositories: public and private. Public repositories allow sharing of container images with the general public, while private repositories restrict access through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or resource-based permissions. Each repository can contain multiple versions of an image, called tagged revisions, which makes it easier to manage different [[api-gateway|stages]] of development.

ECR stores images in a highly available and durable manner by distributing them across multiple Availability Zones (AZs) within a region. It uses Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for object storage and [[dynamodb]] for metadata management. Data is encrypted both at rest and in transit using encryption under the hood.

### [[RDS_Instance_Types|Global Scale Considerations]]

For global distribution of container images, ECR supports cross-region replication. This feature allows automatic copying of images from one region to another, providing low-latency access to container workloads. Replication tasks can be managed via the AWS Management Console, SDKs, CLI, and API.

## Comparison & Anti-Patterns

| Feature                    | ECR                           | Alternatives                              |
| -------------------------- | ----------------------------- | ---------------------------------------- |
| Image hosting             | Docker Hub, Google Container Registry, Azure Container Registry   | *Docker Hub*: less granular control over resources; limited integration with AWS<br>*Google Container Registry*: lower integration with AWS<br>*Azure Container Registry*: lower integration with AWS |
| Multi-account strategy    | ECR Public, Resource Policy, [[organizations|AWS Organizations]]<br>ECR Private per account, AWS Resource Access Manager | *Resource Policy*: requires more manual effort; lacks centralized visibility<br>*AWS Resource Access Manager*: not all resources can be shared yet |
| Quotas                    | Soft limit of 1,000 repos per account | Contact AWS Support to increase the limit |

Common anti-patterns include:

- Using ECR Public for sensitive data: Images hosted publicly can be accessed by anyone, so they should only be used for open-source projects.
- Mixing dev/test/prod environments: Separating these environments ensures better [[appsync|security]] and performance isolation.

## [[appsync|Security]] & Governance

To enforce fine-grained access control, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be applied to restrict actions like `GetImage`, `PutImage`, or `DeleteImage`. Here's a JSON example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ecr:GetDownloadUrlForLayer", "ecr:BatchCheckLayerAvailability"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
          "ecr:BatchGetImage",
          "ecr:GetAuthorizationToken"
      ],
      "Resource": [
        "arn:aws:ecr:*:repository/*image*"
      ]
    }
  ]
}
```

Cross-account access can be granted using either [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or resource-based [[policies]]. For instance, here's how to set up cross-account access using a resource policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccessToImage",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::other_account_number:root"
      },
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchCheckLayerAvailability",
        "ecr:BatchGetImage"
      ],
      "Resource": "arn:aws:ecr:us-west-2:main_account_number:repository/*image*"
    }
  ]
}
```

Additionally, [[organizations]] can apply Service Control [[policies]] (SCPs) to prevent accidental deletion of repositories or unauthorized access.

## Performance & Reliability

ECR has throttling limits on various operations, including pushing and pulling images. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies using AWS SDKs or custom retry mechanisms.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve running ECR repositories in multiple regions and leveraging cross-region replication. By doing so, you ensure minimal downtime during regional disruptions.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for ECR depend on usage metrics like number of repositories, storage size, and number of requests. Calculate costs based on the following formula:

`cost = number_of_repositories × (storage_size + number_of_requests)`

Optimize costs by:

- Removing unused repositories
- Implementing lifecycle [[policies]] to delete older images
- Utilizing smaller base images to minimize storage requirements

## Professional Exam Scenarios

### Scenario 1

Your organization has decided to move their containerized applications to AWS. They want to share specific images between accounts without granting full access to each other's infrastructure. Which approach would best meet their needs?

#### Correct Answer(s):

Use ECR Private along with AWS Resource Access Manager to share individual repositories with specific accounts.

#### Incorrect Answer(s):

Do not use ECR Public, as it does not provide the required level of access control. Do not use ECR Private without proper access management, as it would defeat the purpose of separating the environments.

### Scenario 2

You have been tasked with optimizing ECR costs for your company. The current monthly bill includes $800 for ECR repositories and $200 for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots. How could you reduce overall costs?

#### Correct Answer(s):

Implement lifecycle [[policies]] to remove unused images and old revisions. Additionally, choose smaller base images to minimize storage requirements.

#### Incorrect Answer(s):

Deleting repositories randomly may lead to loss of critical container images. Switching to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots is not recommended because [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots are designed for block-level storage, whereas ECR is optimized for container images.