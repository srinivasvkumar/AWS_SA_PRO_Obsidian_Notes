### Advanced Architecture

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] is a global service that offers centralized management of users, groups, and permissions across all AWS services. It utilizes various components like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, [[policies]], and groups to manage access to AWS resources. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]'s [[RDS_Instance_Types|internals]] involve policy evaluation through the AWS Identity & Access Management Service Specific Condition Keys and Services that Work with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] (e.g., [[sts]], [[cognito]], etc.).

Global scale is achieved by replicating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities within an account and region. However, it does not natively support cross-region replication or redundancy. To address this limitation, you can implement custom solutions using [[lambda]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and [[cloudformation]] [[cloudformation|StackSets]].

### Comparison & Anti-Patterns

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]             | Centralized user & permission management for AWS resources           |
| SSO             | Enterprise SSO involving federated identities (e.g., [[Srinivas_Notes/SAML|SAML]])         |
| [[cognito]] User Pools| Managing mobile app users, OIDC/OAuth based [[api-gateway|authentication]]          |
| [[organizations|AWS Organizations]] | Multi-Account management, [[organizations|Consolidated billing]], Service Control [[policies]] |

Anti-pattern: Hardcoding [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity ARNs in your infrastructure code. Instead, use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and resource-based [[policies]].

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/Environment": "production"
                }
            }
        }
    ]
}
```
Cross-account access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": [
                "sts:AssumeRole"
            ],
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```
Organization SCPs:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyUnapprovedServices",
            "Effect": "Deny",
            "Action": [
                "*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:Service": [
                        "ec2",
                        "elasticloadbalancing",
                        "rds",
                        ...
                    ]
                }
            }
        }
    ]
}
```

### Performance & Reliability

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] has throttling limits on API requests, such as creating/updating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities or assuming roles. If exceeded, you may encounter [[api-gateway|errors]]. Implement exponential backoff strategies when handling such [[api-gateway|errors]].

HA/DR patterns include creating separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities per environment and backing up [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] or custom solutions.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] involve optimizing access to specific resources instead of granting broad permissions. For example, allow `ec2:TerminateInstances` only on instances tagged with a specific key/value pair.

Calculation Example: Assume 100 [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users each having 5 [[policies]] attached. Each policy costs $0.000001/day. The monthly cost would be:

(100 * 5) * ($0.000001 / 30) = $0.001667/month

### Professional Exam Scenarios

Question 1: An organization wants to enforce [[appsync|security]] [[iam|best practices]] while managing multiple AWS accounts. Which AWS services should they leverage?

Correct Answer: [[organizations|AWS Organizations]], AWS SSO, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]].
Justification: [[organizations]] provide [[organizations|consolidated billing]], service control [[policies]], and multi-account management. SSO enables enterprise SSO involving federated identities. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] manages centralized user and permission management for AWS resources.

Incorrect Answer: Using only [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] is insufficient since it doesn't support multi-account management directly.

Question 2: A company needs to implement least privilege access for their developers working on [[ec2]] instances. What approach should they follow?

Correct Answer: Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with the minimum required [[ec2]] instance actions (e.g., Describe, Start, Stop). Attach these roles to relevant groups containing developer users. Implement resource-level permissions using condition keys (e.g., restricting access to instances with specific tags).

Incorrect Answer: Creating individual [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for every developer without grouping them into roles would lead to policy sprawl and maintenance issues.