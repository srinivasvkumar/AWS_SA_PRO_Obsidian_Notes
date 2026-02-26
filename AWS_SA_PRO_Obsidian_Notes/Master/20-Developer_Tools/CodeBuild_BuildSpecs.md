### [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces artifacts. It has a serverless architecture, so it scales automatically without requiring you to manage any infrastructure. The buildspec file is at the heart of CodeBuild, providing [[AWS_SA_PRO_Obsidian_Notes/Master/InfrastructureAsCode (CloudFormation)/02-portable-template/instructions|instructions]] for builds in a declarative, YAML format.

Internally, CodeBuild uses Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] under the hood to run Docker containers as part of the build process. Each build environment consists of an image that contains the necessary tools, runtime, and dependencies specified by your buildspec file. CodeBuild supports multiple programming languages, frameworks, and versions, allowing you to test and deploy applications across various [[cloudformation|stacks]].

Global scale can be achieved using the AWS Region/Edge Locations infrastructure. CodeBuild leverages this network to improve build performance by running them closer to their deployment location. This reduces latency while maintaining [[appsync|security]] through isolated resources within each region.

### [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service               | Use Case                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| CodeBuild             | Automated testing, building container images, and customized environments |
| [[cicd|AWS CodePipeline]]      | Full [[cicd|CI/CD]] pipelines orchestrating multiple services                       |
| [[cicd|AWS CodeDeploy]]        | In-place deployment, application updates, and rollbacks                     |
| AWS [[CodeStar]]          | Simplified developer workflow setup                                         |
| AWS Developer Tools Fusion | Centralize management of permissions, configurations, and reporting    |

Anti-patterns include using CodeBuild for non-build tasks like ad-hoc scripts execution or long-running processes better suited for [[ec2]] instances. Also, avoid using CodeBuild directly with version control systems; instead, opt for full [[cicd|CI/CD]] tools like CodePipeline.

### [[RDS_Instance_Types|3. Security & Governance]]

CodeBuild allows fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for controlling user access. For cross-account access, create a role in the account containing the CodeBuild project and attach an appropriate policy, then assume that role from the other account.

Example JSON policy granting `codebuild:BatchCheckLayerAvailability` permission:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "codebuild:BatchCheckLayerAvailability"
            ],
            "Resource": [
              "*"
            ]
        }
    ]
}
```
Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] provide centralized control over CodeBuild actions across multiple accounts. Enforce resource-level [[policies]] using Deny statements in Organization SCPs.

### [[RDS_Instance_Types|4. Performance & Reliability]]

CodeBuild offers high availability through redundant infrastructure within its regions. Configure build timeout settings and retry behavior based on specific requirements. Implement exponential backoff strategies when encountering intermittent issues during the build process.

For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], ensure critical build projects have backups enabled and stored in another region. CodeBuild supports [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]], which simplifies data protection and retention.

### [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls in CodeBuild include setting build timeout limits and using per-minute [[billing]]. To calculate costs, estimate the number of builds per month, average build duration, and chosen instance type.

Example monthly cost calculation:
```python
((number_of_builds * avg_duration_minutes) / 60) * hourly_rate * number_of_days_in_month
```
Use compute type options judiciously, choosing between Managed and Custom Images based on your needs.

### [[RDS_Instance_Types|6. Professional Exam Scenarios]]

#### Scenario A

An organization wants to enforce company-wide [[policies]] restricting CodeBuild usage only to certain IP ranges. Which approach should they follow?

Correct answer: Create a Service Control Policy ([[SCP]]) in [[organizations|AWS Organizations]] that denies all CodeBuild actions except those originating from allowed IP addresses.

Incorrect answer: Modifying the CodeBuild buildspec file to include allowed IP ranges.

#### Scenario B

An e-commerce company experiences frequent spikes in build requests during holiday seasons. They currently use CodeBuild but face throttling [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] due to insufficient concurrent builds. How can they optimize their existing setup?

Correct answer: Increase the number of concurrent builds by purchasing additional build [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|credits]] or using larger build compute types.

Incorrect answer: Implementing a queue system outside of CodeBuild to regulate build requests.