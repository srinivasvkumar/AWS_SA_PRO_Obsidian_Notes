**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[cloudformation|StackSets]] is a [[cloudformation]] feature that enables creating, updating, or deleting [[cloudformation|stacks]] across multiple accounts and regions with a single operation. It uses a "stack set" as an entity representing a collection of [[cloudformation|stacks]].

Internally, [[cloudformation|StackSets]] leverages Service Control [[policies]] (SCPs) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles to manage cross-account permissions. The StackSet admin account creates a role in the member account, allowing [[cloudformation]] to create/update/delete [[cloudformation|stacks]] on its behalf.

Global scale is achieved by distributing stack instances within each target region. However, there isn't global StackSet management – you must perform operations per region.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Issue                            | [[cloudformation|StackSets]]                | Alternatives                         |
| --------------------------------- | ------------------------ | ------------------------------------ |
| Multi-region support             | Limited regional control  | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/11-migrations/datasync|AWS DataSync]], [[Storage Gateway]]        |
| [[cloudformation|Change sets]]                      | Limited change set usage  | CodePipeline, CodeBuild              |
| [[eventbridge]] events               | Not directly supported   | [[lambda]], [[sns]], [[sqs]]                     |
| [[cloudformation|Custom Resources]]                 | Limited functionality    | [[lambda]], layer [[Git_hub_notes/certified-aws-solutions-architect-professional-main/12-security-and-config/cloudhsm|limitations]]           |

Anti-pattern: Using [[cloudformation|StackSets]] when centralized deployment isn't required. This adds complexity and potential failure points without adding value.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve trust relationships between admin and member accounts. Admin accounts require `iam:CreateServiceLinkedRole` and `organizations:DescribeOrganization` permissions. Member accounts need `cloudformation:CreateStack`, `cloudformation:DeleteStack`, etc., permissions granted via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles assumed by the admin account.

Cross-account access requires setting up a StackSet admin account and configuring SCPs to restrict actions based on organizational units (OUs) or individual accounts.

Example JSON policy granting [[cloudformation]] permissions to [[cloudformation|StackSets]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "cloudformation:UpdateStack",
                "cloudformation:CreateChangeSet",
                "cloudformation:ExecuteChangeSet",
                "cloudformation:DeleteChangeSet"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the number of regions and accounts targeted. For example, if you have 100 regions and 200 accounts, your limit would be 100 * 200 = 20,000 operations. To mitigate throttling issues, implement exponential backoff strategies using AWS SDKs.

For high availability, distribute resources across different AZs within each region. Ensure proper resource configuration, such as load balancers and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances, to achieve desired redundancy levels.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be implemented through tagging resources created by [[cloudformation|StackSets]]. Tagging allows monitoring and managing costs at various levels, e.g., department or project. Calculate costs using the [AWS Pricing Calculator](https://calculator.aws).

**6. Professional Exam Scenario**

Scenario: You are tasked with deploying a multi-region, multi-account solution using [[cloudformation]] [[cloudformation|StackSets]] while minimizing costs.

Question 1: Which strategy provides the best [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]?
A) Create separate [[cloudformation|StackSets]] for each region and account.
B) Use a single StackSet across all regions and accounts.
C) Leverage [[organizations|AWS Organizations]] to apply cost-effective SCPs.
D) Implement tagging on all created resources.

Answer: D) Implement tagging on all created resources.

Justification: While other options may optimize deployment efficiency, they do not directly address cost reduction. Tagging enables tracking and controlling expenditures at various levels.

Question 2: What potential issue might arise from using [[cloudformation|StackSets]] in this scenario?
A) Inadequate throttling limits.
B) Insufficient event support.
C) Limited custom resource functionality.
D) Overcomplicated multi-account deployment.

Answer: C) Limited custom resource functionality.

Justification: [[cloudformation|StackSets]] has limited custom resource functionality due to layer [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] used by [[cloudformation|custom resources]]. This may cause issues when implementing specific use cases requiring [[cloudformation|custom resources]].