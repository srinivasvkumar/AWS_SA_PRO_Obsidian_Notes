### Advanced Architecture

At its core, Service Control [[policies]] (SCPs) are organizational level [[policies]] that provide centralized control over the maximum allowed permissions for all accounts within an [[AWS Organization]]. They are attached to the organization root or Organizational Units (OUs) and can be used to enforce [[control-tower|guardrails]] and [[iam|best practices]] across multiple accounts. SCPs are especially useful in large scale environments where maintaining consistent [[appsync|security]] [[policies]] can become challenging.

SCPs are evaluated after [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], service-specific [[policies]], and other permissions but before resource-based [[policies]]. This evaluation order ensures that SCPs act as a safety net to prevent unwanted actions even when other [[policies]] allow them.

### Comparison & Anti-Patterns

| Service | Use Cases |
| --- | --- |
| [[SCP]] | Centralized control of maximum allowed permissions, enforcing [[control-tower|guardrails]], and [[iam|best practices]] |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] | Fine-grained, user-level permissions |
| Service-Specific [[policies]] (e.g., [[Srinivas_Notes/S3|S3]] Bucket [[policies]]) | Access control specific to a service or resource |
| Resource-Based [[policies]] | Allowing external entities to interact with resources directly |

Anti-patterns include using SCPs for fine-grained permissions, managing individual account permissions instead of using OUs, and creating overly permissive SCPs that do not effectively limit permissions.

### [[appsync|Security]] & Governance

SCPs are written in JSON and follow the same structure as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. To deny a permission, you must explicitly specify it. For example, to restrict [[ec2]] instances from being launched in the `us-west-2` region, you would write:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "ec2:RunInstances",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:Region": "us-west-2"
                }
            }
        }
    ]
}
```
Cross-account access can be enabled by specifying principal and source [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] IDs in the [[SCP]].

### Performance & Reliability

SCPs have no direct impact on performance or reliability. However, they should be designed carefully to avoid unintended throttling limits. If too many principals are affected by a single [[SCP]], it may cause increased API call volumes and potential throttling issues. In such cases, consider redesigning your SCPs or splitting them into smaller [[policies]].

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved through careful design of SCPs and other [[policies]]. By limiting which services and regions are accessible, [[organizations]] can minimize unexpected costs caused by unused or underutilized resources. Calculation examples would depend on the specific use case, but generally involve determining the expected usage of each resource and applying appropriate [[policies]] to limit access only to those required.

### Professional Exam Scenario

#### Scenario 1

Suppose your organization has two teams: Team A and Team B. Both teams require access to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], but Team A needs stricter permissions due to compliance requirements.

Correct answer: Create an [[SCP]] that denies specific actions for Team A and attaches it to their account or OU. Other permissions would be granted through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Incorrect answer: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] alone for both teams is insufficient because they can override each other.

#### Scenario 2

Your organization is concerned about the number of API calls made by [[SCP]] evaluations.

Correct answer: Evaluate your current [[SCP]] design and split it into smaller [[policies]] if necessary. Implement exponential backoff strategies in your applications to handle potential throttling.

Incorrect answer: Disabling [[SCP]] evaluations entirely is not recommended as it weakens your overall [[appsync|security]] posture.