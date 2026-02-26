```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
priority: high
---

# Enhanced Technical Notes: ADR - SCPs vs. IAM Policies vs. Permission Boundaries

## UNDER-THE-HOOD MECHANICS:
### SCP (Service Control Policy) Mechanics
> [!abstract] Exam Tip SCPs are evaluated at the AWS Organizations level, restricting permissions for all accounts in an organization.

### IAM Policies Mechanics
> [!abstract] Exam Tip Managed or inline policies attached directly to IAM roles, users, and groups. Evaluated based on the principle of least privilege (POLP).

### Permission Boundaries Mechanics
> [!abstract] Exam Tip Applied only at the role/user level, setting a ceiling for what an entity can do regardless of other attached IAM policies.

## EXHAUSTIVE FEATURE LIST:
### SCP Features
- `Allow` and `Deny` statements
- Applies at the organization level, affecting multiple accounts
- Can be used for compliance and security enforcement

### IAM Policy Features
- Managed policies (reusable)
- Inline policies (specific to individual roles/users/groups)
- Support for conditions and resource-specific permissions
- Supports versioning and tagging

### Permission Boundary Features
- Strict deny mechanism on a per-entity basis
- Can be attached to IAM users, roles but not groups
- Limits the effective set of permissions an entity can have

## STEP-BY-STEP IMPLEMENTATION:

### SCP Implementation Steps
```sh
[[organizations|aws organizations]] create-policy --content file://policy.json --type SERVICE_CONTROL_POLICY --name policy-name
```
- Attach the SCP to an organizational unit (OU) or specific accounts.

### IAM Policy Implementation Steps
```sh
aws [[00_Master_Links_and_Intro|iam]] create-policy --policy-name MyPolicy --policy-document file://my-policy.json
```
- Attach policies to roles, users, and groups as needed.
  ```sh
  aws [[00_Master_Links_and_Intro|iam]] attach-role-policy --role-name myRole --policy-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:policy/MyPolicy
  ```

### Permission Boundary Implementation Steps
- Define a permission boundary policy document.
  ```sh
  aws [[00_Master_Links_and_Intro|iam]] put-role-permissions-boundary --role-name myRole --permissions-boundary-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:policy/MyBoundaryPolicy
  ```

## TECHNICAL LIMITS (2026):
### SCP Limits
- Maximum number of SCPs per organization: 50,000 (adjustable via support request)
- Maximum size for a single SCP: 32 KB

### IAM Policy Limits
- Maximum inline policies per user/role/group: 10
- Maximum managed policies per role/user: 10

### Permission Boundary Limits
- Only one permission boundary can be attached to an entity.
- Maximum size for a single policy document: 6 KB

## CLI & API GROUNDING:
- AWS CLI Commands:
  ```sh
  [[organizations|aws organizations]] create-policy --content file://policy.json --type SERVICE_CONTROL_POLICY --name policy-name
  ```
  ```sh
  aws [[00_Master_Links_and_Intro|iam]] attach-role-policy --role-name myRole --policy-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:policy/MyPolicy
  ```
  ```sh
  aws [[00_Master_Links_and_Intro|iam]] put-role-permissions-boundary --role-name myRole --permissions-boundary-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:policy/MyBoundaryPolicy
  ```

## PRICING & TCO:
- SCPs and IAM policies are part of the AWS Organizations service, which has no additional cost beyond standard usage.
- Permission boundaries do not incur extra costs.

## COMPETITOR COMPARISON:
### SCP vs. IAM Policies
> [!abstract] Exam Tip SCPs enforce governance at a higher level (organization), while IAM policies provide granular access control within individual accounts.

### IAM Policies vs. Permission Boundaries
> [!abstract] Exam Tip IAM policies define what permissions an entity can have, whereas permission boundaries limit the maximum set of those permissions.

## CONTEXTUALIZATION WITHIN THE AWS ECOSYSTEM:
- SCPs integrate with AWS Organizations to enforce cross-account controls.
- IAM policies interact seamlessly with various AWS services like S3, RDS, and VPC for granular access control.
- Permission boundaries are often used in conjunction with roles for temporary credentials within secure environments.

## REAL-WORLD USE CASES & EXAMPLES:
### SCP Use Case
> [!abstract] Exam Tip Enforce a policy across all accounts to prevent deletion of critical resources (e.g., RDS instances).

### IAM Policy Use Case
> [!abstract] Exam Tip Define fine-grained permissions for developers to access only specific S3 buckets and EC2 instances.

### Permission Boundary Use Case
> [!abstract] Exam Tip Limit the maximum permissions an assumed role can have in a cross-account scenario.

## ARCHITECTURAL DIAGRAMS:
- Diagram showing SCPs applied at the organization level, with IAM policies attached to roles/users within each account.
  ```
  [Organization] -> [SCP]
         |              |
      [OU]           [Account A] -> [IAM Policies] -> [Role/User]
                   /                /
                 [Account B] -> [IAM Policies] -> [Role/User]
  ```

## EXAM TRAPS & COMMON MISCONCEPTIONS:
### Misunderstanding SCP Evaluation
> [!danger] Distractor SCPs are evaluated after IAM policies, meaning they can override permissions.

### Confusing Managed vs. Inline Policies
> [!danger] Distractor Managed policies are reusable across multiple entities, while inline policies are specific to a single entity.

### Permission Boundaries Scope
> [!danger] Distractor Permission boundaries apply strictly on roles/users but not groups.

## DISTRACTOR ANALYSIS:
### Scenario 1: RPO/RTO Requirements
- If an organization has stringent Recovery Point Objective (RPO) and Recovery Time Objective (RTO), SCPs might not be the right tool since they do not address backup or disaster recovery mechanisms.
- **Most Correct Logic**: Use other AWS services like AWS Backup for cross-account backups.

### Scenario 2: Cost Optimization
- For cost optimization, IAM policies and permission boundaries are more granular and can be tailored to reduce unnecessary resource access, whereas SCPs are broader and more focused on compliance.
- **Most Correct Logic**: Optimize policies and permissions at the role/user level.

### Scenario 3: Multi-Account Management
- In a scenario where multiple accounts need different levels of control, using only SCPs might lead to overly restrictive or permissive environments across all accounts.
- **Most Correct Logic**: Use a combination of SCPs and IAM policies tailored to each account's needs.

## THE "MOST CORRECT" LOGIC:
- **Operational Excellence**: IAM policies and permission boundaries are more suitable for operational excellence as they allow granular control over access.
- **Cost Optimization**: Using permission boundaries can help in cost optimization by limiting excessive permissions which might lead to unnecessary resource usage.

## ENTERPRISE GOVERNANCE:
### AWS Organizations
> [!abstract] Exam Tip Use SCPs within AWS Organizations to enforce compliance and security across multiple accounts.

### SCP & IAM Policies Integration
> [!abstract] Exam Tip Leverage SCPs for cross-account controls and IAM policies for granular access management.

### RAM (Resource Access Manager)
> [!abstract] Exam Tip Utilize RAM for sharing resources across different accounts while ensuring proper permission boundaries are set.

### CloudTrail Auditing
> [!abstract] Exam Tip Set up CloudTrail to monitor API activity for auditing purposes, which can be used in conjunction with SCPs and IAM policies.

## TIER-3 TROUBLESHOOTING:
### KMS Key Policy Conflicts
During cross-account migrations, ensure KMS key policy permissions are correctly set to avoid conflicts that could lead to inaccessible encrypted resources.
> [!abstract] Exam Tip Review the order of policy application and ensure no explicit deny statements in IAM policies override SCPs.

### SCP Evaluation Issues
Troubleshoot SCP evaluation issues by reviewing the order of policy application and ensuring no explicit deny statements in IAM policies override SCPs.
> [!abstract] Exam Tip Check SCP and IAM policy configurations to ensure they align with intended controls.

## DECISION MATRIX FORMAT
| Feature/Scenario        | SCP                | IAM Policy               | Permission Boundary |
|------------------------|--------------------|--------------------------|---------------------|
| Cross-Account Control   | High (Organization)| Medium (Per Account)     | Low                 |
| Granularity             | Coarse             | Fine                     | Ceiling on IAM      |
| Common Use Case         | Compliance, Governance| Access to Resources| Secure Role Usage  |

## INFORMATION RELEVANT TO THE SAP-C02 EXAM:
- **SCP vs. IAM Policy**: Understanding when to use SCPs for organizational control versus IAM policies for account-level access.
- **Permission Boundaries**: Knowing how permission boundaries can enforce security in multi-account environments.

## REFERENCE MATERIALS:
1. [AWS Organizations Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_scp.html)
2. [IAM Policy and Access Management](https://aws.amazon.com/iam/)
3. [Permission Boundaries for IAM Users, Roles, and Groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)

By following this comprehensive guide, SAP-C02 candidates can thoroughly understand the nuances between SCPs, IAM policies, and permission boundaries, making informed decisions in exam scenarios.

## Related Notes
- [AWS Organizations Best Practices](related-notes/aws_organizations_best_practices.md)
- [IAM Policies in Depth](related-notes/iam_policies_in_depth.md)
- [Permission Boundaries Explained](related-notes/permission_boundaries_explained.md)
```