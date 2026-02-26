```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[cicd|CI/CD]] and Deployment Pipelines

#### UNDER-THE-HOOD MECHANICS:
1. **Internal Data Flow**: [[Continuous Integration]] involves automatically running tests, compiling code, and validating changes to the source code repository. The data flows from version control systems like [[GitHub]], or [[AWS CodeCommit]], through build steps using tools like [[Jenkins]], [[AWS CodeBuild]], or [[GitHub Actions]].
2. **Consistency Models**: [[cicd|CI/CD]] pipelines ensure consistency by enforcing a set of checks (linting, unit tests) that must pass before changes are merged into the main branch and then deployed to staging/production environments.
3. **Underlying Infrastructure Logic**: Deployment pipelines use infrastructure as code tools like [[AWS CloudFormation]], or [[Terraform]], to manage infrastructure configuration. They orchestrate the deployment process using services like [[AWS CodePipeline]], which triggers builds, tests, and deployments based on predefined rules.

#### EXHAUSTIVE FEATURE LIST:
1. **Source Control Integration**: Supports multiple version control systems.
2. **Build Automation**: Automated building of applications.
3. **Test Automation**: Running unit, integration, and functional tests.
4. **Deployment Automation**: Automatic deployment to various environments (dev, staging, prod).
5. **Approval Gates**: Manual approval steps before moving from one stage to another.
6. **Rollback Mechanisms**: Automated rollback in case of failures.
7. **[[vpc-flow-logs|Logging]] & Monitoring**: Detailed logs for every step and monitoring using services like [[Amazon CloudWatch]].
8. **[[appsync|Security]] Scanning**: Integration with [[appsync|security]] tools for code analysis.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Setup Source Control Repository**: Initialize repository on GitHub, [[cicd|AWS CodeCommit]], etc.
2. **Configure Build Steps**: Define build configuration files (Dockerfiles, Jenkinsfile).
3. **Integrate Tests**: Setup unit and integration tests in the pipeline.
4. **Define Deployment [[api-gateway|Stages]]**: Use [[CodePipeline]] to define different [[api-gateway|stages]] like build, test, deploy.
5. **Setup Approval Gates**: Configure manual approval steps as needed.
6. **Monitor and Log**: Set up [[cloudwatch]] for monitoring and [[vpc-flow-logs|logging]].

#### TECHNICAL LIMITS (2026):
1. **Soft Quotas**:
   - Number of concurrent builds in CodeBuild: 20
   - Number of pipelines per AWS account: 1000
2. **Hard Quotas**:
   - Maximum pipeline size in [[CodePipeline]]: 5MB

#### CLI & API GROUNDING:
- **AWS CLI Commands**: `aws codepipeline create-pipeline`, `aws codebuild start-build`
- **API Flags**: `create_pipeline`, `start_build`

#### PRICING & TCO:
1. **Cost Drivers**:
   - Compute resources used in builds and deployments.
   - Storage for artifacts.
2. **Optimization Strategies**:
   - Use [[00_Master_Links_and_Intro|spot instances]] for build jobs.
   - Optimize storage with lifecycle [[policies]].

#### COMPETITOR COMPARISON:
- **[[cicd|AWS CodePipeline]] vs GitHub Actions**: CodePipeline is more tightly integrated with AWS services, whereas [[GitHub Actions]] offers broader ecosystem support but may require more manual integration.
- **Trade-offs**:
  - Ecosystem [[api-gateway|integrations]]: GitHub Actions have a wider range of plugins and actions for different ecosystems.
  - Cost-effectiveness: GitHub Actions can be free up to certain limits.

#### INTEGRATION:
1. **[[cloudwatch]]**: For [[vpc-flow-logs|logging]] and monitoring pipeline activities.
2. **[[00_Master_Links_and_Intro|IAM]]**: Define roles with least privilege access to different AWS services used in pipelines.
3. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Configure [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] for secure communication between [[CodePipeline]], [[CodeBuild]], etc.

#### USE CASES:
1. **Enterprise Example**: A financial institution using [[cicd|CI/CD]] to deploy a trading platform across multiple environments (dev, staging, prod) with rigorous testing and [[appsync|security]] checks at each stage.
2. **Real-world Scenario**: An e-commerce company implementing automated rollbacks in case of failed deployments during peak shopping seasons.

#### DIAGRAMS:
- Placeholder for architectural diagram showing integration between [[CodeCommit]], [[CodeBuild]], [[CodePipeline]], [[cloudformation]], and [[cloudwatch]].

> [!abstract] Exam Tip
Identify scenarios where [[cicd|CI/CD]] services like CodePipeline or CodeBuild might be a wrong answer. For instance, if extensive infrastructure automation beyond code deployment is required, using [[00_Master_Links_and_Intro|AWS CloudFormation]] and [[AWS CDK]] would be more appropriate.

> [!check] Implementation
For small applications with simple manual deployments, tools like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or direct uploads through the AWS Management Console might suffice over setting up a complete [[cicd|CI/CD]] pipeline.

> [!warning] Quota
Ensure all quotas, pricing, and features are aligned with the AWS updates by 2026. Reference the latest official documentation for accuracy.

#### DECISION MATRIX:
| Features                 | Use Cases                   |
|--------------------------|------------------------------|
| Source Control Integration| GitOps workflows            |
| Build Automation         | Automated software builds    |
| Test Automation          | Continuous testing           |
| Deployment Automation    | Zero-downtime deployments    |
| Approval Gates           | Governance and compliance    |

#### 2026 UPDATES:
- Ensure all quotas, pricing, and features are aligned with the AWS updates by 2026.
- Reference latest official documentation for accuracy.

> [!danger] Distractor
Common misconception: Thinking that [[cicd|CI/CD]] pipelines are only for code changes; they can also manage infrastructure as code. Misunderstanding the difference between [[api-gateway|stages]] in a pipeline (build vs deploy).

#### EXAM SCENARIOS:
1. **Scenario**: A company needs to deploy microservices in a high-security environment.
   - **Answer**: Use [[CodePipeline]], [[CodeBuild]], and [[00_Master_Links_and_Intro|IAM]] roles with least privilege access to ensure [[appsync|security]] compliance.

> [!abstract] Exam Tip
Explain trade-offs for performance vs cost. For instance, CodeBuild offers scalable build infrastructure but may become expensive as complexity increases.

#### GROUNDING & VALIDATION:
Reference official AWS documentation and align content with the latest exam blueprint for SAP-C02.

#### ARCHITECTURAL TRADE-OFFS:
Impact of design choices such as choosing [[CodePipeline]] over other [[cicd|CI/CD]] tools on overall deployment reliability, [[appsync|security]], and cost-efficiency.