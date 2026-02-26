**[[RDS_Instance_Types|1. Advanced Architecture]]**

At its core, AWS Cloud9 is a cloud-based integrated development environment (IDE) that allows developers to write, run, and debug code in a browser. It supports many programming languages, including Python, JavaScript, and Java, and provides a seamless experience by integrating with AWS services such as [[CodeStar]], [[CodeCommit]], and CodeBuild.

Internally, Cloud9 uses containers to provide an isolated and secure environment for each user's development work. Each container runs a customized Amazon Linux AMI and includes tools and packages required for specific programming languages. The containers are managed by the AWS [[Fargate]] service, which enables Cloud9 to automatically scale up or down based on demand.

Global scale is achieved through the use of Amazon Virtual Private Cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) peering, which allows Cloud9 instances to connect to resources across different regions. This enables low-latency connections between Cloud9 and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]], even when they are located in different regions.

When using Cloud9 with multiple accounts, it is recommended to use [[organizations|AWS Organizations]] to create a hierarchy of accounts and apply Service Control [[policies]] (SCPs) to enforce [[appsync|security]] and compliance [[policies]]. Additionally, Cloud9 can be configured to use AWS Single Sign-On (SSO) to enable users to sign in to Cloud9 using their corporate credentials.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

The following table compares Cloud9 with alternative services:

| Service | Ideal Use Case |
| --- | --- |
| Cloud9 | Development environment for AWS services |
| AWS [[CodeStar]] | End-to-end software development workflow |
| [[cicd|AWS CodeBuild]] | Continuous integration and delivery |
| [[cicd|AWS CodeCommit]] | Version control for source code |

Common anti-patterns for Cloud9 include:

* Using Cloud9 as a general-purpose IDE instead of a development environment for AWS services.
* Not configuring appropriate permissions and access controls, leading to potential [[appsync|security]] risks.
* Not monitoring usage and costs, resulting in unexpected charges.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Cloud9 may involve allowing or denying access to specific AWS resources, such as Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets or Amazon [[ec2]] instances. For example, the following JSON policy allows a user to access a specific Cloud9 environment:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloud9:CreateEnvironmentMember",
                "cloud9:DeleteEnvironmentMember",
                "cloud9:UpdateEnvironmentMember"
            ],
            "Resource": "arn:aws:cloud9:us-east-1:123456789012:environment:my-env",
            "Condition": {
                "StringEquals": {
                    "cloud9:permissiontype": "readwrite"
                }
            }
        }
    ]
}
```
Cross-account access can be enabled by creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the target account and allowing the Cloud9 service to assume the role. The role should have permissions to perform the necessary actions on the relevant resources.

Organization SCPs can be used to enforce [[appsync|security]] [[policies]] at the organizational level. For example, an [[SCP]] can be created to deny access to all AWS services except for Cloud9 and a few other essential services.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits in Cloud9 include a maximum of five environments per region and a limit of 50 concurrent sessions per environment. To avoid throttling issues, it is recommended to implement exponential backoff strategies when making API requests to Cloud9.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns for Cloud9 can be achieved by using multiple regions and implementing backup and restore mechanisms. For example, a Cloud9 environment can be backed up to an Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in another region, and the environment can be restored from the backup in case of a failure.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls for Cloud9 can be implemented by setting usage limits, alarms, and [[Budgets]]. Usage limits can be set to prevent unexpected charges, while alarms and [[Budgets]] can be used to notify users when costs exceed a certain threshold.

Calculation examples for Cloud9 costs include:

* Environment creation and deletion: $0.0116 per hour
* Concurrent sessions: $0.0000016 per session per minute
* Amazon [[ec2]] instance usage: Based on the type and duration of the instance

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
A company has multiple AWS accounts and wants to use Cloud9 to develop and test [[lambda|AWS Lambda]] functions. How would you configure Cloud9 to allow developers to access the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] in a centralized location?

Correct answer:
Configure [[organizations|AWS Organizations]] to create a hierarchy of accounts and apply SCPs to enforce [[appsync|security]] and compliance [[policies]]. Create a Cloud9 environment in one of the accounts and configure it to use AWS SSO to enable developers to sign in to Cloud9 using their corporate credentials. Set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering to connect the Cloud9 environment to the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] hosting the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

Incorrect answer:
Create a Cloud9 environment in each account and grant developers access to each environment individually. This approach lacks centralization and scalability.

Scenario 2:
A developer is experiencing throttling issues when using Cloud9 to develop and test an application. What steps can be taken to mitigate the issue?

Correct answer:
Implement exponential backoff strategies when making API requests to Cloud9. Monitor usage and adjust throttling limits accordingly. Consider using a separate AWS account for development and testing to avoid conflicts with production workloads.

Incorrect answer:
Ignore the throttling issues and continue developing and testing the application. This approach may result in [[api-gateway|errors]] and delays in the development process.