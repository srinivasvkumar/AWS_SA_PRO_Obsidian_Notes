## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[config|AWS Config]] is a service that records configuration items (CI) of your resources and allows you to evaluate these CIs against custom rules ([[config]] Rules). The following sections cover advanced architecture aspects such as multi-account strategies, [[RDS_Instance_Types|internals]], [[RDS_Instance_Types|global scale considerations]], and 'Under the hood' mechanics.

### Multi-Account Strategies

For managing multiple accounts in an organization, you can use [[organizations|AWS Organizations]] and its features like Service Control [[policies]] (SCPs) and AWS Delegated Administrator. With [[organizations|AWS Organizations]], you can create a hierarchy of organizational units (OUs) and assign SCPs to them. This way, you can enforce centralized control over configurations across all accounts within an OU.

To aggregate [[config]] Rules evaluation results from various member accounts, you can use [[config|AWS Config]] Conformity Score. It provides a single view of compliance status in your [[AWS Organization]].

### [[RDS_Instance_Types|Internals]] and Global Scale Considerments

[[config|AWS Config]] uses Amazon [[cloudwatch]] Events to record configuration changes continuously. Each time a resource change occurs, [[config|AWS Config]] creates a new Configuration Item (CI) containing metadata about the resource. These CIs are stored in the corresponding [[config|AWS Config]] region's [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. To achieve global scale, [[config|AWS Config]] has a central regional hub called the "[[config|AWS Config]] Rule Evaluation Service" that manages cross-region evaluations for [[config]] Rules.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

Comparing [[config|AWS Config]] with other services like [[parameter-store|AWS Systems Manager Parameter Store]] and [[security-hub|AWS Security Hub]], here are some anti-patterns and guidelines when not to use [[config]]:

| Service | Anti-pattern / Alternative |
|---|---|
| [[config|AWS Config]] | Not suitable for storing sensitive data or application settings. For these cases, consider using [[parameter-store|AWS Systems Manager Parameter Store]]. |
| [[config|AWS Config]] | Shouldn't be used for threat detection or aggregating findings from multiple [[appsync|security]] services. In these scenarios, prefer [[security-hub|AWS Security Hub]]. |

Common design mistakes include creating too many custom rules, which may lead to increased costs and complexity. Instead, leverage pre-built managed rules whenever possible.

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve fine-grained permissions to manage [[config|AWS Config]] resources. Here's a JSON example demonstrating permission to perform specific actions on [[config]] Rules:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "config-rule:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access can be achieved by setting up an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account that grants required permissions and then assumes it from the destination account. Additionally, you can use Organization SCPs to restrict or allow specific [[config]] actions at scale.

## [[RDS_Instance_Types|4. Performance & Reliability]]

[[config|AWS Config]] imposes throttling limits on requests per second to prevent overwhelming the service. To handle such [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies using AWS SDKs. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns should follow [[iam|best practices]] for each integrated service since [[config]] itself doesn't offer native [[dr]] capabilities.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls involve understanding how much storage and API calls will cost based on usage. For instance, [[config]] stores configuration items for six years by default, but you can adjust the [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] to reduce costs. Also, consider deleting old or unnecessary [[config]] Rules to minimize API call expenses.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1:

Suppose you need to track configuration changes in a large enterprise with multiple AWS accounts. How would you implement a solution using [[config|AWS Config]] and [[organizations|AWS Organizations]]?

Correct Answer:

1. Create an [[AWS Organization]] and set up OUs.
2. Add relevant AWS accounts to the Organization.
3. Enable [[config|AWS Config]] in each member account.
4. Apply Service Control [[policies]] (SCPs) to limit or allow specific [[config]] actions.
5. Aggregate [[config]] Rules evaluation results using [[config|AWS Config]] Conformity Score.

Incorrect Answers:

1. Implementing [[config|AWS Config]] in one central account only—this wouldn't work well for multi-account environments.
2. Using [[parameter-store|AWS Systems Manager Parameter Store]] instead of [[config]]—Parameter Store does not provide [[config]] change tracking functionality.

### Scenario 2:

Imagine you have a strict budget for [[config|AWS Config]] usage due to high API call volume. What measures could you take to optimize costs without compromising governance requirements?

Correct Answer:

1. Reduce [[config]] Rules' number if they aren't critical for governance.
2. Adjust [[config]] Rules' parameters to minimize API calls (e.g., less frequent evaluations).
3. Lower the [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] for Configuration Items storage.

Incorrect Answers:

1. Disabling [[config|AWS Config]] entirely—it's essential for maintaining governance standards.
2. Migrating to another service like [[parameter-store|AWS Systems Manager Parameter Store]]—it cannot replace [[config]]'s functionality.