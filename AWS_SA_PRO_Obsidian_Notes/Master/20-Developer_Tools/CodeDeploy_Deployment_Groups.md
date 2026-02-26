## Advanced Architecture

CodeDeploy Deployment Groups are used to deploy code across multiple instances or targets in various AWS Regions or Availability Zones (AZs) within a Region. They allow for granular control over deployment settings, rollback options, and alarms. The internal workings of CodeDeploy involve several components:

- **Application:** Represents what is being deployed, such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], Amazon [[ecs]] services, or [[ec2]] instances.
- **Deployment Group:** Represents a set of targets (e.g., [[ec2]] instances, On-premises servers, or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]) that will receive the application revision during a deployment.
- **Deployment Configurations:** Define how a deployment should proceed, including traffic routing, rollback behavior, and [[route53|health checks]].
- **Deployment Triggers:** Automatically start a deployment when an event occurs, like pushing new code to GitHub or an Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket.

When working at scale, there are some considerations for Deployment Groups:

1. **Global Deployments:** To achieve global scale, you can create separate Deployment Groups per Region and manage them independently. This approach requires additional management overhead but allows for maximum flexibility.
2. **Multi-Account Strategies:** For multi-account environments, you can centrally manage your CodeDeploy resources using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs). Alternatively, you can use AWS [[AppRunner]], which supports cross-account deployments natively.

## Comparison & Anti-Patterns

| Service                 | Suitable Use Cases                                                                   | Anti-Patterns                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| CodeDeploy             | Application updates on [[ec2]] instances, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], and [[ecs]] services      | Not suitable for container image updates in Kubernetes clusters; use CodeCommit/CodeBuild instead |
| [[cicd|AWS CodePipeline]]       | End-to-end [[cicd|CI/CD]] pipelines spanning multiple services                            | Overkill for simple application deployments; use CodeCommit/CodeBuild/CodeDeploy directly    |
| AWS [[CodeStar]]           | Simplified development environment setup and project management                | Limited customizability; not designed for advanced deployment scenarios              |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]]     | Infrastructure as code                                                          | Not intended for application deployment; use CodeCommit/CodeBuild/CodeDeploy instead        |
| AWS Elastic Beanstalk  | Managed platform for applications                                               | Limited control over underlying infrastructure; not ideal for custom deployments          |

## [[appsync|Security]] & Governance

To secure Deployment Groups, you must configure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and permissions. Here's an example JSON policy granting `codedeploy:CreateDeploymentGroup` permission to a user:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "codedeploy:CreateDeploymentGroup",
            "Resource": "arn:aws:codedeploy:*:*:deploymentgroup:*"
        }
    ]
}
```

For cross-account access, you can either assume roles or use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users with appropriate permissions. Moreover, you can enforce centralized governance using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs). An example [[SCP]] would be:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "codedeploy:CreateDeploymentGroup",
            "Resource": "arn:aws:codedeploy:*::*",
            "Condition": {
                "StringNotEqualsIgnoreCase": {
                    "aws:PrincipalOrgID": "ORG_ID"
                }
            }
        }
    ]
}
```

This [[SCP]] denies any `CreateDeploymentGroup` actions outside the specified ORG\_ID.

## Performance & Reliability

CodeDeploy has throttling limits and error handling mechanisms. By default, you can have up to 25 concurrent deployments per region. If you encounter throttling [[api-gateway|errors]], implement exponential backoff strategies to ensure reliable operation.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your Deployment Groups across multiple AZs and regions. Implement blue/green deployments to minimize downtime and reduce risk.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for CodeDeploy come from understanding usage patterns and optimizing accordingly. Some methods include:

- Limiting the number of active Deployment Groups
- Using smaller instance sizes for deployment targets
- Minimizing unnecessary retries by implementing error handling and exponential backoff strategies

Calculate costs using the [AWS Pricing Calculator](https://calculator.aws/#/) and monitor spending using AWS [[billing|Cost Explorer]].

## Professional Exam Scenarios

### Scenario 1: Multi-Region Deployment

Suppose you need to deploy a Node.js web application to multiple AWS Regions while minimizing latency and ensuring high availability. Which solution meets these requirements?

Correct answer: Create separate Deployment Groups per Region and use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] to direct users to the closest available endpoint.

Incorrect answer: Use a single Deployment Group and enable global rollouts.

Explanation: Global rollouts do not provide the same level of latency reduction as having separate Deployment Groups per Region.

### Scenario 2: Securing CodeDeploy

You want to restrict CodeDeploy usage to specific AWS accounts within your organization. What combination of AWS services achieves this goal?

Correct answer: Use [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to limit CodeDeploy actions to designated AWS accounts.

Incorrect answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] allowing only specific IP addresses to perform CodeDeploy actions.

Explanation: While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can restrict specific IP addresses, they cannot limit actions to particular AWS accounts.