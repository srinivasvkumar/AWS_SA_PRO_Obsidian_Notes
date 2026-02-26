```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[AWS Config]] Custom Rules & Remediation

#### Under-the-Hood Mechanics:
[[AWS Config]] Custom Rules allow users to define custom logic using [[Lambda functions]] to evaluate the compliance of resource configurations. When a new configuration item (CI) is created or updated, [[config|AWS Config]] triggers these rules to check if the CI complies with specified [[cloudformation|conditions]].

1. **Data Flow**:
   - [[config|AWS Config]] captures and stores resource metadata.
   - Custom rules are triggered by changes in this metadata.
   - [[lambda]] function associated with the custom rule evaluates the compliance status.
   - Compliance results are stored back into [[AWS Config]].

2. **Consistency Models**:
   - **Synchronous**: Rules can be set to evaluate immediately upon configuration change.
   - **Asynchronous**: Batch evaluation runs at scheduled intervals (e.g., every 15 minutes).

3. **Underlying Infrastructure Logic**:
   - Configuration items are stored in [[Amazon S3]] buckets for durability and availability.
   - [[00_Master_Links_and_Intro|Lambda functions]] execute the custom logic and return compliance status to [[config|AWS Config]].

#### Exhaustive Feature List
- Custom rule evaluation using [[Lambda functions]]
- Support for multiple programming languages (Node.js, Python, Java)
- Batch and real-time evaluations
- Integration with [[AWS Config]] managed rules
- Remediation actions linked to non-compliant resources

#### Step-by-Step Implementation:
1. **Define the Compliance Condition**:
   - Identify what specific resource configurations should be checked.

2. **Create a [[lambda]] Function**:
   - Write and deploy a [[lambda]] function that evaluates the compliance status.

3. **Set Up Custom Rule in [[config|AWS Config]]**:
   - Configure the rule to trigger the [[lambda]] function on relevant configuration changes.

4. **Configure Remediation Actions (Optional)**:
   - Define actions to automatically correct non-compliant resources.

#### Technical Limits
- 200 custom rules per account (soft limit)
- Each [[lambda]] function has execution limits based on [[lambda|AWS Lambda]] quotas
- Custom rule evaluation can take up to 15 minutes for batch evaluations

#### CLI & API Grounding:
- **AWS CLI**:
  ```sh
  aws configservice put-config-rule --config-rule file://rule.json
  ```

- **API Flags**:
  - `PutConfigRule`
  - `StartConfigRulesEvaluation`
  - `DescribeRemediationExecutionStatus`

#### Pricing & TCO:
- Custom rules are billed based on the usage of [[AWS Lambda]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].
- [[00_Master_Links_and_Intro|Cost optimization]] strategies include optimizing [[lambda]] execution times and minimizing unnecessary evaluations.

#### Competitor Comparison
- **Azure Policy**: Azure provides similar functionality but uses [[Azure Functions]] instead of [[lambda]]. Azure Policy has built-in support for more automatic remediation options.
- **GCP Organization [[policies]]**: GCP’s [[policies]] are defined in JSON format, offering a different approach to configuration management and compliance.

#### Integration:
- **[[cloudwatch]]**: Custom rules can trigger [[cloudwatch]] events based on non-compliance status.
- **[[00_Master_Links_and_Intro|IAM]]**: [[00_Master_Links_and_Intro|IAM]] roles provide permissions for [[00_Master_Links_and_Intro|Lambda functions]] and [[config|AWS Config]] access.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: [[00_Master_Links_and_Intro|Lambda functions]] can be executed within VPCs, ensuring network isolation.

#### Use Cases
1. **[[appsync|Security]] Compliance**: Ensure that only compliant [[security groups]] are used across the organization.
2. **Cost Management**: Monitor usage of expensive resources like [[00_Master_Links_and_Intro|RDS]] instances or [[00_Master_Links_and_Intro|EBS]] volumes.
3. **Operational Efficiency**: Automatically remediate misconfigurations in [[00_Master_Links_and_Intro|EC2]] instance tags or [[00_Master_Links_and_Intro|IAM]] [[policies]].

#### Decision Matrix
| Feature            | [[appsync|Security]] Compliance | Cost Management | Operational Efficiency |
|--------------------|---------------------|-----------------|------------------------|
| Custom Rules       | High                | Medium          | Low                    |
| Remediation        | High                | High            | Medium                 |

#### Exam Traps
> [!danger] Distractor
Misunderstanding the difference between managed rules and custom rules.
Forgetting to set up [[00_Master_Links_and_Intro|IAM]] roles correctly for [[00_Master_Links_and_Intro|Lambda functions]].

#### 2026 Updates:
- Expected improvements in the integration of [[AWS Config]] with other services like [[appsync|Security]] Hub.
- Potential increase in default limits for custom rules and [[lambda]] execution.

#### Exam Scenarios
> [!abstract] Exam Tip
Given a scenario, select the appropriate remediation action for a non-compliant resource.
Identify steps to troubleshoot a misconfigured [[lambda]] function used by an [[config|AWS Config]] rule.

#### Interaction Patterns:
- **Service-to-service**: Configuration items are stored in [[Amazon S3]] and evaluated by [[00_Master_Links_and_Intro|Lambda functions]] triggered by [[config|AWS Config]] events.
- **Data flow**: Changes in configuration items trigger evaluations which update compliance status, potentially triggering remediation actions.

> [!warning] Quota
Technical limits include 200 custom rules per account (soft limit) and execution limits based on [[lambda|AWS Lambda]] quotas.