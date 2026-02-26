## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, Audit Manager is a service that helps you manage audits of your AWS resources and applications. It provides a central location to create customized audits, automate evidence collection, and archive results. Understanding its internal workings is crucial when designing solutions at scale.

### Components

- **Audit Dashboard**: Provides an overview of all audits, their progress statuses, and findings.
- **Audit Reports**: Generate reports based on audit findings and share them securely.
- **Evidence Collection Rules**: Define specific criteria for collecting evidence from AWS services.
- **Integration with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS Services]]**: Leverages [[config|AWS Config]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], and [[cloudwatch]] Events for data collection.

### [[RDS_Instance_Types|Global Scale Considerations]]

Audit Manager operates within a single region but can be used across multiple accounts in an organization using Service Control Policy ([[SCP]]) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. For global scale requirements, implement [[organizations|AWS Organizations]] to centrally manage member accounts and apply SCPs consistently.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

Comparing Audit Manager to alternative services like [[config|AWS Config]] or [[security-hub|AWS Security Hub]]:

| Service | Primary Function | Advantage | Disadvantage |
|---|---|---|---|
| Audit Manager | Manage audits and evidence collection | Centralized dashboard, customizable audits | Limited to predefined templates |
| [[config|AWS Config]] | Monitor infrastructure changes | Customizable rules and resource relationships | Lacks built-in audit management features |
| [[security-hub|AWS Security Hub]] | Aggregate [[appsync|security]] alerts and automate remediation | Integrates with multiple AWS services | May require additional tools for custom audits |

Anti-patterns include using Audit Manager as a standalone tool without integrating it with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as [[config|AWS Config]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], and [[cloudwatch]] Events. This could lead to incomplete evidence collection and limited visibility into resource configurations and activities.

## [[RDS_Instance_Types|3. Security & Governance]]

Implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] by leveraging JSON policy documents. Here's an example of granting Audit Manager access to a user in another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "auditmanager:CreateAudit",
                "auditmanager:Get*",
                "auditmanager:List*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpce": "vpce-1234abcd"
                }
            }
        }
    ]
}
```

Cross-account access can be established using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {}
        }
    ]
}
```

Use Organization SCPs to enforce [[control-tower|guardrails]] and restrict actions that may impact audit trails or introduce [[appsync|security]] risks.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Audit Manager has throttling limits on API calls. To avoid exceeding these limits, implement exponential backoff strategies in case of failed requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute Audit Manager usage across multiple regions by setting up separate audits for each region. Ensure proper replication of required data sources (e.g., [[config|AWS Config]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]) in each target region.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls can be achieved through tagging resources related to Audit Manager, such as [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], and [[cloudwatch]] Events. Calculate costs using the AWS Pricing Calculator or the [[billing|Cost Explorer]] tool.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

**Scenario 1:** Your company needs to perform regular audits of its AWS resources and applications across multiple accounts. The audits should follow specific guidelines and collect evidence automatically. Which AWS services would best meet these requirements?

Correct answer: [[audit-manager|AWS Audit Manager]], [[config|AWS Config]], AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], and AWS [[cloudwatch]] Events.

Explanation:

- Audit Manager allows creating customized audits and automating evidence collection.
- [[config]] monitors resource changes and generates compliance reports.
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] records API calls made on AWS resources for further analysis.
- [[cloudwatch]] Events triggers rules based on specific events, enabling automatic evidence collection.

Incorrect answers:

- [[security-hub|AWS Security Hub]] does not provide built-in support for custom audits.
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Firewall Manager]] focuses on managing firewalls and network [[appsync|security]] [[policies]] rather than audit management.

**Scenario 2:** A customer wants to ensure that only authorized users have access to Audit Manager APIs in their AWS environment. How can they achieve this while minimizing potential [[appsync|security]] risks?

Correct answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Service Control [[policies]] (SCPs) to control access to Audit Manager APIs and prevent unauthorized changes in the environment.

Explanation:

- Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that allow or deny specific actions based on user roles.
- Attach these [[policies]] to relevant [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles.
- Implement SCPs to enforce [[control-tower|guardrails]] and restrict actions that may impact audit trails or introduce [[appsync|security]] risks.

Incorrect answers:

- Trust [[policies]] alone cannot limit access to Audit Manager APIs since they only define who can assume a role, not which actions are allowed.
- Using only identity-based [[policies]] might not be sufficient because they do not address cross-account access restrictions or organizational-level constraints.