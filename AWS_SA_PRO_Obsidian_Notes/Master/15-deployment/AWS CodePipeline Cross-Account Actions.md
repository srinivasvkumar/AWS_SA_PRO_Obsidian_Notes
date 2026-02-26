```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[AWS CodePipeline]] Cross-Account Actions

#### Under-the-Hood Mechanics:
[[AWS CodePipeline]] allows you to create and manage continuous integration and deployment pipelines, enabling automated workflows across different [[api-gateway|stages]]. When using cross-account actions, CodePipeline uses [[00_Master_Links_and_Intro|IAM]] roles and permissions to allow an action in one account to interact with services or resources in another AWS account.

1. **Data Flow:** The pipeline runs in the source account (where it is created), but certain actions can be configured to execute in a different target account.
2. **Consistency Models:** CodePipeline ensures consistency by using [[00_Master_Links_and_Intro|IAM]] roles and cross-account permissions, ensuring that each step of the pipeline has the necessary access to resources in other accounts.
3. **Underlying Infrastructure Logic:** The infrastructure leverages [[AWS Identity and Access Management (IAM)]] for [[api-gateway|authentication]] and authorization across different AWS accounts.

#### Exhaustive Feature List:
- **Cross-Account Actions:** Execute actions in a different account than where the pipeline is created.
- **Service Role Permissions:** Each action can be associated with a service role that grants permissions to access resources in the target account.
- **[[00_Master_Links_and_Intro|IAM]] Roles and [[policies]]:** Use cross-account roles for granular control over what resources an action can interact with.
- **Resource Access Management ([[ram]]):** Optionally, you can use [[AWS Resource Access Manager (RAM)]] to share resources across accounts within your organization.

#### Step-by-Step Implementation:
1. **Setup [[00_Master_Links_and_Intro|IAM]] Roles:**
   - Create a service role in the source account that CodePipeline will assume.
   - In the target account, create an [[00_Master_Links_and_Intro|IAM]] role and trust policy for cross-account execution.
2. **Configure Cross-Account Actions:**
   - Define actions in CodePipeline to specify the resource ARN in another account.
3. **Set Up Resource Permissions:**
   - Ensure the service role has permissions to access necessary resources in both accounts.

#### Technical Limits:
- **Soft Quotas:** Typically, you can have multiple pipelines but may face [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] on concurrent executions and [[00_Master_Links_and_Intro|step functions]].
- **Hard Quotas (2026):** [[cicd|AWS CodePipeline]] supports up to 19 [[api-gateway|stages]] per pipeline. You are limited by the number of actions per stage and the size of your JSON definition.

#### CLI & API Grounding:
- **CLI Commands:** Use `aws codepipeline create-pipeline` or `update-pipeline` with the appropriate action configuration.
    ```bash
    aws codepipeline update-pipeline --cli-input-json file://pipeline.json
    ```
- **API Flags:**
    - Action configuration for cross-account actions includes `RoleArn`, specifying the ARN of the [[00_Master_Links_and_Intro|IAM]] role in the target account.

#### Pricing & TCO:
- **Cost Drivers:** Pricing is based on the number and duration of pipeline executions.
- **Optimization Strategies:** Use AWS [[billing|Cost Explorer]] to monitor usage, enable [[billing]] alerts, and utilize reserved capacity for [[00_Master_Links_and_Intro|cost optimization]].

#### Competitor Comparison:
- **Azure DevOps Pipelines:** Offers similar capabilities but may require more manual setup for cross-account actions.
- **GitHub Actions with GitHub Enterprise Server:** Requires additional configuration to manage permissions across accounts but provides flexibility in pipeline management.

#### Integration:
- **[[cloudwatch]]:** CodePipeline integrates seamlessly with [[cloudwatch]] for [[vpc-flow-logs|logging]] and monitoring.
- **[[00_Master_Links_and_Intro|IAM]]:** Utilizes [[00_Master_Links_and_Intro|IAM]] roles extensively for access control.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:** Can be configured to run actions within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], enhancing [[appsync|security]].

#### Use Cases:
- **Multi-Account Environments:** Enterprises managing multiple AWS accounts can use cross-account pipelines to manage deployments across environments securely.
- **Resource Isolation:** Teams in different departments may need access to resources managed by another team’s account.

#### Diagrams:
> Placeholder for architectural diagram showing pipeline execution flow across two AWS accounts with detailed service interactions and [[00_Master_Links_and_Intro|IAM]] roles. The diagram should highlight the interaction between source and target accounts, emphasizing [[appsync|security]] boundaries and data flows.

#### Exam Traps:
- **Misconceptions:** Not all actions can be cross-account; certain services require specific permissions or configurations.
- **Complex Configurations:** Be cautious about overcomplicating pipeline [[api-gateway|stages]] that involve cross-account interactions without proper [[00_Master_Links_and_Intro|IAM]] setup.

#### Decision Matrix: Features vs. Use Cases
| Feature                  | Use Case                     |
|--------------------------|------------------------------|
| Cross-Account Actions    | Multi-account environments   |
| Service Role Permissions | Granular access control      |

#### 2026 Updates:
- Ensure alignment with AWS’s updated [[00_Master_Links_and_Intro|IAM]] [[policies]] and roles for cross-account actions.
- Account for any new features or API changes introduced in CodePipeline by 2026.

#### Exam Scenarios:
> [!abstract] Exam Tip
Design a pipeline that builds an application in one account and deploys it to another, ensuring proper role configuration and access management.

#### Interaction Patterns:
Service-to-service interactions primarily involve [[00_Master_Links_and_Intro|IAM]] roles for cross-account actions. The service role in the source account assumes permissions defined by the target account’s trust policy.

> [!warning] Quota
- **Soft Quotas:** Typically, you can have multiple pipelines but may face [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] on concurrent executions and [[00_Master_Links_and_Intro|step functions]].
- **Hard Quotas (2026):** [[cicd|AWS CodePipeline]] supports up to 19 [[api-gateway|stages]] per pipeline. You are limited by the number of actions per stage and the size of your JSON definition.

#### Strategic Alignment:
Ensuring thorough understanding of CodePipeline configurations and [[00_Master_Links_and_Intro|IAM]] roles is critical for passing SAP-C02 exams.

> [!danger] Distractor
- **Scenario 1 - Using [[sns]] for Cross-Account Communication**: While AWS [[sns]] can be used to trigger actions across accounts, it does not provide the integrated [[cicd|CI/CD]] workflow that CodePipeline offers.
- **Scenario 2 - Manual Deployment Across Accounts**: Manually deploying artifacts across different accounts bypasses automation and introduces human error, reducing reliability.
- **Scenario 3 - Using [[lambda|AWS Lambda]] for Cross-Account Triggers**: Although possible, [[00_Master_Links_and_Intro|Lambda functions]] would require additional configuration to handle cross-account permissions and setup compared to CodePipeline's integrated capabilities.
- **Scenario 4 - Using [[step-functions|AWS Step Functions]] for Orchestration**: While powerful for complex workflows, [[00_Master_Links_and_Intro|Step Functions]] lack the specialized [[cicd|CI/CD]] features of CodePipeline and may result in more manual effort.

> [!check] Implementation
1. **Setup [[00_Master_Links_and_Intro|IAM]] Roles:**
   - Create a service role in the source account that [[CodePipeline]] will assume.
   - In the target account, create an [[00_Master_Links_and_Intro|IAM]] role and trust policy for cross-account execution.
2. **Configure Cross-Account Actions:**
   - Define actions in CodePipeline to specify the resource ARN in another account.
3. **Set Up Resource Permissions:**
   - Ensure the service role has permissions to access necessary resources in both accounts.

> [!warning] Quota
- [[kms]] Key Policy Conflicts: Incorrect [[kms]] key [[policies]] can prevent actions from accessing encrypted artifacts in different accounts. Ensure the appropriate [[00_Master_Links_and_Intro|IAM]] roles have permissions to decrypt using specified keys.
- [[00_Master_Links_and_Intro|IAM]] Role Mismatch: Misconfigured [[00_Master_Links_and_Intro|IAM]] roles or role chaining across accounts can lead to permission [[api-gateway|errors]], requiring manual intervention to correct.
- Network Isolation Issues: Cross-account actions might fail if proper network access and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] configurations are not in place, especially with private subnets.

> [!check] Implementation
- **CLI Commands:** Use `aws codepipeline create-pipeline` or `update-pipeline` with the appropriate action configuration.
    ```bash
    aws codepipeline update-pipeline --cli-input-json file://pipeline.json
    ```
- **API Flags:**
    - Action configuration for cross-account actions includes `RoleArn`, specifying the ARN of the [[00_Master_Links_and_Intro|IAM]] role in the target account.

#### Decision Matrix: Use Case vs. Solution Table

| Use Case                  | Solution                                 |
|---------------------------|------------------------------------------|
| Automated Deployment       | [[cicd|AWS CodePipeline]]                         |
| Manual Trigger Across Accounts | SNS/Manual Scripts                     |
| Complex Orchestration      | [[step-functions|AWS Step Functions]]                       |
| Lambda-based Automation    | [[lambda|AWS Lambda]] + [[api-gateway|API Gateway]]                 |

#### Recent Updates
- **GA Features (2026)**:
    - Enhanced cross-account actions in CodePipeline, including improved [[00_Master_Links_and_Intro|IAM]] role management.
    - New [[cloudformation]] integration features that simplify deployment and rollback processes.
- **Deprecated Items**:
    - Legacy cross-account APIs for [[00_Master_Links_and_Intro|IAM]] roles might be deprecated, necessitating updates to the latest practices.