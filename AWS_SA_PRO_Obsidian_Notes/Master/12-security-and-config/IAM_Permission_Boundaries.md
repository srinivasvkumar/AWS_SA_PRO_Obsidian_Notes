### [[RDS_Instance_Types|1. Advanced Architecture]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies|Permission Boundaries]] operate at the upper limit of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] hierarchy, allowing fine-grained control over what permissions an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity can have. They act as a "blueprint" for creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and enforce least privilege principle across accounts. A permission boundary doesn't provide any permissions by itself but restricts what actions are allowed when used in conjunction with other [[policies]].

Key components include:

- **Permission boundary document**: A JSON policy that defines the maximum allowable permissions for an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity or role. It can be attached to users, groups, or roles.
- **Boundary evaluation logic**: The internal mechanism responsible for evaluating whether a given request conforms to the defined boundaries. If not, it denies access even if explicitly granted by another policy.

Global scale is achieved through centralized management within an organization and leveraging Service Control [[policies]] (SCPs) to apply consistent restrictions across all member accounts.

### [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Permissions       | Centralized, hierarchical control over [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]                   |
| SCPs                  | Broadest level of control for [[organizations]]; applies to all resources  |
| Service Control Planes | Specific services like [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]]; lower granularity          |

Anti-patterns:

- Using [[policies|permission boundaries]] as standalone [[policies]] instead of combining them with other [[policies]].
- Applying [[policies|permission boundaries]] directly to entities without using groups or roles.
- Creating excessively permissive boundaries that negate their purpose.

### [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] require careful consideration of nested [[cloudformation|conditions]], explicit deny statements, and cross-account access rules. For example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["ec2:*"],
            "Resource": ["*"]
        },
        {
            "Effect": "Deny",
            "NotAction": ["ec2:DescribeInstances"],
            "Resource": ["*"]
        }
    ]
}
```

Cross-account access should follow the principle of least privilege, while taking into account potential circular dependencies between trust relationships.

Organization SCPs offer an additional layer of [[appsync|security]] by enforcing constraints on actions performed within member accounts. For instance:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyUnapprovedServices",
            "Effect": "Deny",
            "Action": "*.*",
            "NotResource": [
                "arn:aws:service:*:*:*/*
            ]
        }
    ]
}
```

### [[RDS_Instance_Types|4. Performance & Reliability]]

There are no specific throttling limits related to [[policies|permission boundaries]]. However, general [[iam|best practices]] such as exponential backoff strategies during high error rates still apply.

HA/DR patterns involve maintaining up-to-date copies of permission boundary documents in multiple regions or accounts in case of failure. This can be automated using tools like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] or [[AWS CDK]].

### [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls with [[policies|permission boundaries]] mainly revolve around optimizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] usage. By setting appropriate boundaries, [[organizations]] can prevent unnecessary API calls and minimize potential costs associated with excessive permissions.

Calculation examples would depend on specific use cases and cannot be provided generically.

### [[RDS_Instance_Types|6. Professional Exam Scenarios]]

Scenario 1: An organization wants to ensure that its developers cannot accidentally delete [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets containing critical data. Which approach would you recommend?

Correct answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies|permission boundaries]] limiting the `s3:DeleteBucket` action to only authorized personnel. Incorrect answers might suggest using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]] or object-level permissions directly on the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] resources.

Scenario 2: A company has multiple AWS accounts managed under a single [[AWS Organization]]. They want to enforce a [[appsync|security]] policy prohibiting the creation of [[ec2]] instances with unrestricted internet access. How could they achieve this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies|permission boundaries]]?

Correct answer: Create an Organization [[SCP]] denying relevant `ec2:AuthorizeSecurityGroupIngress` and `ec2:ModifyInstanceAttribute` actions for affected [[appsync|security]] groups or instances. Incorrect answers might propose using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user [[policies]] or service control [[policies]] that are too broad or don't address the issue effectively.