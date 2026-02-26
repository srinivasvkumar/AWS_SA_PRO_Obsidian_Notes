## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] is a fully managed [[rds|data security]] and privacy service that uses machine learning to discover, classify, and protect sensitive data in AWS. It operates at a deep level of granularity, analyzing metadata associated with data stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], including object-level metadata, user-defined tags, and access control settings.

[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]]'s architecture consists of several components:

- **Data Lake**: A centralized repository composed of an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and [[dynamodb|DynamoDB tables]] used to store and manage metadata ingested from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] resources.
- **Classification Engine**: A rule-based system that identifies sensitive information based on predefined or custom classification rules.
- **Notification Service**: An event-driven component responsible for sending alerts when sensitive data is detected.
- **User Interface**: A web-based console that provides visibility into [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie findings]] and allows users to configure and manage their environment.

[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] also supports cross-region data replication, enabling [[organizations]] to create a single, consolidated view of their sensitive data across multiple regions. This feature leverages the Global Suppression List, which prevents duplicates from being reported as findings.

## Comparison & Anti-Patterns

| Service | Ideal Use Case |
|---|---|
| [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] | Large-scale, high-granularity analysis of sensitive data in Amazon [[Srinivas_Notes/S3|S3]] |
| [[GuardDuty]] | Network activity monitoring and threat detection in AWS environments |
| [[Collab_Notes_detailed/Security/Detective|Detective]] | Forensic investigation and manual analysis of [[appsync|security]] events |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] as a standalone tool for network-level threat detection. [[GuardDuty]] should be used instead for this purpose.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created using JSON snippets like this one, allowing fine-grained control over who can perform specific actions within [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "macie2:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpce": "vpce-12345678"
                }
            }
        }
    ]
}
```

Cross-account access can be established by configuring appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. For example, a role in Account A could have a policy granting `macie2:Describe*`, `macie2:Get*`, and `macie2:List*` permissions, while a role in Account B has a trust relationship allowing the first role to assume it.

Organization SCPs can enforce centralized management of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] settings through Service Control [[policies]] (SCPs) like this one:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyMacieCreation",
            "Effect": "Deny",
            "Action": [
                "macie2:Create*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalOrgID": "o-123456789012"
                }
            }
        }
    ]
}
```

## Performance & Reliability

[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] imposes throttling limits on certain API calls, such as `CreateJob`, `CreateMember`, and `UpdateFindingsFilter`. To avoid exceeding these limits, implement exponential backoff strategies that gradually increase the time between retries.

HA/DR patterns involve deploying [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] in multiple regions and configuring cross-region data replication. This ensures continued operation in case of a regional outage and enables data protection across geographically dispersed infrastructure.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented by using AWS [[billing|Cost Explorer]] to analyze usage trends and identify potential optimizations. Additionally, you can enable [[billing]] alerts that trigger when your costs exceed a defined threshold.

For example, if you want to track monthly spending on [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]], set up an alert like this:

1. Navigate to the AWS [[billing]] Console.
2. Click "Bills" and then "Create Alert".
3. Select "Monthly" as the time period.
4. Enter a name, threshold value, and email address.
5. Choose "[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]]" as the service.
6. Save the alert.

## Professional Exam Scenarios

### Vignette 1

Scenario: Your organization stores sensitive financial information in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. You need to ensure all instances of this data are classified correctly and receive timely notifications when unauthorized access occurs.

Correct Answer: Implement [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] to automatically detect and classify the sensitive data, and send notifications upon detection.

Incorrect Answer: Rely solely on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]] and user-defined tags to restrict access and monitor resource usage.

Justification: While [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]] and user-defined tags provide some level of access control and resource tracking, they do not offer the same degree of automation and intelligence as [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]]. By implementing [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]], you can continuously monitor your [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets, detect any instances of sensitive financial information, and receive real-time notifications when unauthorized access occurs.

### Vignette 2

Scenario: Your company manages multiple AWS accounts and wants to centrally manage [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] settings and prevent individual users from disabling the service.

Correct Answer: Implement an Organization [[SCP]] to deny [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] creation in each account.

Incorrect Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants specific users permission to invoke [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] APIs.

Justification: The Organization [[SCP]] approach ensures centralized management of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] settings and prevents individual users from disabling the service. Conversely, creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants specific users permission to invoke [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] APIs does not effectively prevent them from disabling the service. Instead, this method only governs which users can interact with the [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Macie]] API, potentially leaving other avenues open for disabling the service.