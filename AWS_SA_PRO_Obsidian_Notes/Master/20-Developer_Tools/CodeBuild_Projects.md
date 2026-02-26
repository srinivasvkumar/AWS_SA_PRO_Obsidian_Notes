### Advanced Architecture

At its core, CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces artifacts. It has several key components:

- **Build Environment**: A combination of an operating system, programming language runtime, and tools. CodeBuild provides preconfigured build environments for popular languages and [[eb|platforms]].
- **Buildspec file**: A YAML or JSON file in the source code root directory that defines the build phases, commands, and [[Artifact]] actions.
- **Build project**: Represents a single build run. Contains settings like the build environment, build spec, timeouts, and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] configuration.

Global scale is achieved through regional endpoints and [[api-gateway|caching]] mechanisms. CodeBuild stores build artifacts in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and can be configured to use an optional build cache stored in Amazon Elastic Container Registry (ECR) or Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. CodeBuild also supports container-based build environments, enabling you to bring your own custom build images.

### Comparison & Anti-Patterns

| Service                      | When to Use                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| CodeBuild                    | Automated builds, testing, and deployments; short-lived containers   |
| CodePipeline                | Full [[cicd|CI/CD]] pipelines spanning multiple services; release workflows       |
| CodeDeploy                   | In-place deployments; blue/green and rolling updates                     |
| [[CodeStar]] Connections         | Integrating development tools; not recommended for production workloads |
| AWS Developer Tools - AWS CLI | Manual command line operations; not recommended for automated tasks    |

Anti-patterns include using CodeBuild as a general-purpose compute service, running long-lived processes, and storing large amounts of data within build environments.

### [[appsync|Security]] & Governance

CodeBuild supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and roles. For cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account allowing necessary actions and attach it to a service role in the destination account. Additionally, apply Service Control [[policies]] (SCPs) at the organization level to enforce specific restrictions.

Example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codebuild:BatchCheckBuild",
        "codebuild:StartBuild"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

### Performance & Reliability

CodeBuild offers throttling limits based on the number of concurrent builds and build minutes per region. To manage these limits, implement exponential backoff strategies when encountering throttling [[api-gateway|errors]].

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute resources across multiple regions and ensure proper backup and restore procedures. Consider using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]], [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]], or custom solutions for this purpose.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls are available by setting build timeout limits, using shared build projects, and utilizing [[cost-allocation-tags|cost allocation tags]]. Calculate costs using the formula:

Cost = (Minutes used × Minute rate) + (Build data storage × Storage rate)

### Professional Exam Scenario 1

Suppose you need to set up a multi-account continuous integration and deployment solution for a large enterprise. The organization has strict [[appsync|security]] requirements, including centralized control over developer accounts and resource usage. Which services would you choose?

Correct answer: CodeBuild, CodePipeline, and [[CodeCommit]] integrated with [[organizations|AWS Organizations]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]], and AWS Single Sign-On (SSO). This setup allows for centralized management while ensuring developers have only the permissions they require.

Incorrect answer: Using [[CodeStar]] Connections with external development tools. While [[CodeStar]] Connections simplifies tool integration, it does not provide the same level of centralized control and visibility required by the organization.

### Professional Exam Scenario 2

Your company needs to automate building and testing Docker images for a web application. They want to minimize operational overhead and optimize costs. Which approach should you recommend?

Correct answer: Use CodeBuild with container-based build environments, pulling the base image from Amazon ECR. Configure the buildspec file to perform the build, test, and publish steps. Store the final Docker image in Amazon ECR, leveraging granular cost controls.

Incorrect answer: Running the build process manually using the AWS CLI. While this approach might work for initial testing, it lacks scalability and automation capabilities, leading to increased operational overhead and potential consistency issues.