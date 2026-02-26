```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[AWS CodeDeploy]] Blue-Green and Canary Deployments

#### UNDER-THE-HOOD MECHANICS:

- **[[Blue-Green Deployment]]**: This strategy deploys the new version of your application to a separate environment ([[Green]]) while keeping the old version running in another environment ([[Blue]]). Once validated, traffic is switched from [[Blue]] to [[Green]].
  - **Consistency Models**: Ensures no downtime by maintaining two parallel environments. 
  - **Underlying Infrastructure Logic**: Uses [[Auto Scaling Groups]] and [[Elastic Load Balancers]] for managing instances and routing traffic.

- **[[Canary Deployment]]**: A subset of the total traffic is routed to a new version while most traffic continues to flow to the old version ([[Blue]]). If the canary environment proves successful, traffic is fully redirected.
  - **Consistency Models**: Allows gradual validation by only exposing a portion of users to changes at first.
  - **Underlying Infrastructure Logic**: Similar to Blue-Green but with weighted routing and traffic shifting logic.

#### EXHAUSTIVE FEATURE LIST:

- **Blue-Green Deployment**:
  - Supports multiple deployment types (In-place, Replacement)
  - Rollback support
  - Traffic management via load balancers
  - Health checks on new instances before switching traffic

- **Canary Deployment**:
  - Gradual traffic shifting
  - Canary percentages can be defined
  - Custom health conditions for monitoring the canary environment
  - Automatic or manual traffic routing

#### STEP-BY-STEP IMPLEMENTATION:

1. Define your deployment group in [[AWS CodeDeploy]].
2. Create a new revision with updated application code.
3. Initiate the Blue-Green/Canary deployment from the AWS Management Console, CLI, or API.
4. Monitor deployment progress through [[CloudWatch]] logs and metrics.

#### TECHNICAL LIMITS (As of 2026):

- **Blue-Green Deployment**:
  - Maximum number of deployment groups per account: 100
  - Maximum instances in a deployment group: Varies based on region and instance type

- **Canary Deployment**:
  - Minimum canary percentage: 5%
  - Maximum canary percentage: 95%
  - Health check timeout: Up to 6 hours per health check

#### CLI & API GROUNDING:

- AWS CLI Command for Blue-Green deployment:
  ```bash
  aws deploy create-deployment --application-name <app_name> \
    --deployment-group-name <group_name> \
    --revision file://<path_to_revision>.zip \
    --deployment-config-name CodeDeployBlueGreen
  ```

- API Reference: `CreateDeployment` with `CodeDeployBlueGreen` for blue-green, and appropriate health conditions for canary.

#### PRICING & TCO:

- **Pricing**: Pay per instance-hour used in the deployment.
- **Optimization Strategies**:
  - Use auto-scaling to manage costs based on traffic patterns.
  - Utilize reserved instances or savings plans for cost predictability.

#### COMPETITOR COMPARISON:

- **Azure DevOps Release Pipelines**: Offers similar blue-green and canary strategies but with different integrations and monitoring tools.
- **Google Cloud Deployment Manager**: Uses a declarative method to manage deployments but lacks the automated health checks of CodeDeploy.

#### INTEGRATION:

- **CloudWatch**: Logs and metrics for monitoring deployment progress and health checks.
- **IAM**: Roles and permissions are critical for CodeDeploy actions.
- **VPC**: Network isolation can be maintained using VPC configurations.

#### USE CASES:

- **Retail Website Updates**: Rolling out new features with minimal impact on users, ensuring quick rollback if needed.
- **Financial Services**: Implementing canary releases to test changes in a low-risk environment before full-scale deployment.

#### DIAGRAMS:

- Placeholder for Blue-Green Deployment diagram
  - ![Blue-Green Diagram](#)
  
- Placeholder for Canary Deployment diagram
  - ![Canary Diagram](#)

> [!abstract] Exam Tip:
Misunderstanding the difference between blue-green and canary deployments.
Overlooking the need for appropriate health checks in both deployment types.

#### DECISION MATRIX:

| Feature          | Blue-Green Deployment      | Canary Deployment        |
|------------------|----------------------------|--------------------------|
| Health Checks    | Mandatory                  | Optional                 |
| Traffic Splitting| Binary (full switch)       | Gradual                  |
| Rollback Support | Automatic                  | Manual                   |

#### 2026 UPDATES:

Ensure alignment with the latest AWS documentation and release notes for any updates or new features.

#### EXAM SCENARIOS:

- Scenario: A company wants to update their application without downtime. Which deployment strategy should be used?
- Answer: Blue-Green Deployment

> [!danger] Distractor:
Incorrect Use Case: Application with External Dependencies
If your application has significant external dependencies (e.g., databases, third-party services), a sudden switch in traffic may lead to issues if those dependencies aren't prepared for the new version.

#### INTERACTION PATTERNS:

- **Service-to-service interaction**: CodeDeploy interacts with [[ec2]], Auto Scaling, and CloudWatch for health checks and monitoring.

> [!check] Implementation:
AWS CLI Command for Blue-Green deployment:
```bash
aws deploy create-deployment --application-name <app_name> \
  --deployment-group-name <group_name> \
  --revision file://<path_to_revision>.zip \
  --deployment-config-name CodeDeployBlueGreen
```

#### STRATEGIC ALIGNMENT:

Focus on the detailed mechanics of deployments as they are critical exam topics.

#### PROFESSIONAL TONE:

Ensure explanations are concise yet comprehensive, focusing on actionable insights.

#### AUDIENCE:

Tailored specifically to SAP-C02 candidates, highlighting exam-relevant details.

#### PRIORITIZATION:

Emphasize high-weight topics such as deployment strategies and best practices for monitoring.

#### CLARITY:

Use clear examples and diagrams to illustrate complex concepts effectively.

#### GROUNDING:

Reference official AWS documentation such as the [[CodeDeploy User Guide]](https://docs.aws.amazon.com/codedeploy/latest/userguide/).

#### VALIDATION:

Align content with the latest SAP-C02 exam blueprint for accuracy.

#### ARCHITECTURAL TRADE-OFFS:

Trade-offs include increased cost due to maintaining two environments in Blue-Green versus potential risk of canary failures.
```