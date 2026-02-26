## Advanced Architecture

At its core, [[security-hub|AWS Security Hub]] is a regional service that aggregates [[appsync|security]] checks from multiple AWS services, such as Amazon [[GuardDuty]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Inspector]], and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Amazon Macie]]. It provides a comprehensive view of an organization's [[appsync|security]] posture by normalizing and consolidating findings across accounts and regions.

[[appsync|Security]] Hub operates at the global level using AWS Regions. Each Region contains a copy of [[appsync|Security]] Hub data, providing geographic redundancy. However, [[appsync|Security]] Hub does not support cross-Region replication or sharing. Data transfer between Regions requires explicit integration and duplication efforts.

Internally, [[appsync|Security]] Hub uses [[lambda|AWS Lambda]] functions and Amazon Simple Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]) to implement its functionality. The underlying architecture includes a centralized [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket per Region, which stores all customer findings in JSON format. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] process incoming [[appsync|security]] events and store them as findings in the associated [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. These functions also convert third-party findings into [[appsync|Security]] Hub's native format before storing them.

## Comparison & Anti-Patterns

| Feature               | [[appsync|Security]] Hub                   | Alternatives                                                         |
| --------------------- | ------------------------------- | -------------------------------------------------------------------- |
| Aggregation           | Centralizes findings              | [[config|AWS Config]], AWS [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], AWS [[cloudwatch]] Events                |
| Native [[api-gateway|Integrations]]    | [[GuardDuty]], [[Collab_Notes_detailed/Security/Inspector|Inspector]], [[Collab_Notes_detailed/Security/Macie|Macie]]     | Custom [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], AWS [[eventbridge]]                             |
| Third-Party [[api-gateway|Integrations]] | CrowdStrike, JupiterOne, Sumo Logic | Custom [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], AWS [[eventbridge]], APIs                       |
| Scalability          | Built-in                        | Requires custom solutions based on [[lambda|AWS Lambda]], [[api-gateway|API Gateway]], and [[Git_hub_notes/certified-aws-solutions-architect-professional-main/16-other/kendra|others]] |

Anti-pattern: Using [[appsync|Security]] Hub solely for log aggregation without [[appsync|security]] context. This could lead to unnecessary costs and increased complexity without adding value to [[appsync|security]] monitoring.

## [[appsync|Security]] & Governance

[[appsync|Security]] Hub supports complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] through fine-grained permissions. For example, you can create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to control specific actions like `AcceptInvitation`, `DeclineInvitation`, and `UpdateOrganizationConfiguration`.

Cross-account access is possible via [[organizations|AWS Organizations]] and [[appsync|Security]] Hub's integration with AWS Single Sign-On (SSO). To enable these features, attach the appropriate managed [[policies]] to relevant [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or users.

[[appsync|Security]] Hub supports Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]]. By creating custom SCPs, administrators can enforce [[appsync|security]] [[policies]] across member accounts.

### Sample [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "securityhub:BatchImportFindings",
                "securityhub:BatchDeleteFindings"
            ],
            "Resource": "*"
        }
    ]
}
```

## Performance & Reliability

[[appsync|Security]] Hub has throttling limits on various API calls. For instance, the `AcceptInvitation` action allows up to five requests per minute. To handle such [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], apply exponential backoff strategies when making repeated calls.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve deploying [[appsync|Security]] Hub in multiple Regions. Since [[appsync|Security]] Hub does not natively support cross-Region sharing, duplicate findings may occur. Consider implementing custom solutions to deduplicate findings and minimize potential confusion.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

[[appsync|Security]] Hub charges based on the number of findings ingested, billed in increments of 100. To optimize costs, set up granular cost controls using AWS [[billing|Cost Explorer]]. Specifically, monitor [[appsync|Security]] Hub usage and allocate [[Budgets]] to individual teams or projects.

Additionally, consider configuring automated deletion of old findings to avoid unnecessary storage costs. Implement lifecycle management [[policies]] to remove outdated findings after a specified [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]].

## Professional Exam Scenarios

**Scenario 1:** An organization wants to integrate [[appsync|Security]] Hub with a third-party [[appsync|security]] tool that generates findings in JSON format. The solution should automatically convert third-party findings to [[appsync|Security]] Hub's native format.

Correct answer: Implement a custom [[lambda]] function triggered by an event bridge rule. This function will parse incoming third-party findings in JSON format and convert them to [[appsync|Security]] Hub's native format.

Incorrect answer: Create an [[glue|AWS Glue]] job to transform third-party findings. This approach is incorrect because it introduces additional complexity and higher costs compared to the [[lambda]] function method.

**Scenario 2:** A multi-account organization seeks to centralize its [[appsync|security]] findings using [[appsync|Security]] Hub while enforcing [[appsync|security]] [[policies]] across all accounts.

Correct answer: Enable [[appsync|Security]] Hub within the organization's master account and invite other member accounts. Attach required [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Service Control [[policies]] (SCPs) to ensure proper access and policy enforcement.

Incorrect answer: Deploy [[appsync|Security]] Hub in each member account and configure cross-account sharing. This approach is incorrect because it lacks centralized management and policy enforcement capabilities offered by [[appsync|Security]] Hub in conjunction with [[organizations|AWS Organizations]].