## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval is designed for long-term archival storage of infrequently accessed data. It provides expedited retrieval of archived objects within seconds while maintaining the same durability and high-throughput capabilities as other [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]. This is achieved through several key components:

- **Primary Storage**: Objects are stored in geographically distributed Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets across multiple Availability Zones (AZs) for resiliency.
- **Metaurl Indexing**: Metadata about each object is indexed separately from the actual object data, enabling faster retrieval times.
- **Warm Storage Tier**: A portion of the archive is kept in a warm storage tier for instant retrievals.
- **Data Durability**: Data is redundantly stored within and across multiple facilities.

## Comparison & Anti-Patterns

| Service                 | Ideal Use Case                                                              |
| ----------------------- | ------------------------------------------------------------------------ |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval | Occasional retrievals with predictable access patterns; low-cost archiving |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval | Fast retrievals with unpredictable access patterns; higher costs          |
| [[Srinivas_Notes/S3|S3]] Intelligent-Tiering   | Automatic [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] based on object access frequency             |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval when [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval would suffice due to infrequent access.

## [[appsync|Security]] & Governance

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy example granting read-only access to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "glacier:DescribeJob",
                "glacier:GetJobOutput",
                "glacier:ListJobs"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "glacier:ArchiveDescription": [
                        "MyGlacierVault",
                        "InstantRetrievalAccessPolicy"
                    ]
                }
            }
        }
    ]
}
```

### Cross-Account Access

To enable cross-account access, perform these steps:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with necessary permissions.
2. Attach the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] vault or bucket.
3. In the destination account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user with required actions.
4. Configure trust relationships between the two accounts using AssumeRole.

### Organization SCPs

Use Organization Service Control [[policies]] (SCPs) to enforce specific restrictions across member accounts, e.g., preventing accidental deletion of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval archives:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDeleteFromGlacier",
      "Effect": "Deny",
      "Action": [
        "glacier:DeleteArchive",
        "glacier:DeleteVault",
        "glacier:DeleteVaultNotifications",
        "glacier:RemovePermission"
      ],
      "Resource": [
        "arn:aws:glacier::*:vault/*"
      ],
      "Condition": {
        "StringNotEqualsIgnoreCase": {
          "aws:PrincipalArn": "arn:aws:iam::<source_account>:root"
        }
      }
    }
  ]
}
```

## Performance & Reliability

### Throttling Limits

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval has the following throttling limits:

- Maximum concurrent restore requests per vault: 5
- Maximum number of initiated jobs per second per AWS account: 20

Implement exponential backoff strategies to handle potential throttling [[api-gateway|errors]] during high-volume restores.

### HA/DR Patterns

For HA/DR scenarios, replicate data across regions using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] or [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]]. For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes, store retrieval jobs in another region using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]].

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be applied by setting up [[billing]] alerts based on usage thresholds and utilizing tag-based cost allocation. Calculate costs using the [AWS Pricing Calculator](https://calculator.aws).

Example monthly cost breakdown for 100 TB of data stored with 10% restoration rate:

- Storage: $1,666.67 (assuming US East (N. Virginia))
- Restorations: $933.33 (assumes 10% restoration rate at $0.01 per GB)
- Total: $2,600

## Professional Exam Scenarios

### Scenario 1

You are tasked with designing a solution that stores large volumes of medical imaging data for long-term retention while ensuring fast retrieval times. The solution must comply with strict [[appsync|security]] requirements, including data encryption and access control. Which [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] storage class should you choose?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval
Justification: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval offers fast retrieval times suitable for medical imaging data while adhering to [[appsync|security]] requirements such as data encryption and access control through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Incorrect Answers:

- [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard: Not suited for long-term retention due to higher costs.
- [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA: Not recommended for production workloads requiring high availability.

### Scenario 2

Your organization needs to implement a multi-account strategy for storing backups across different departments. Each department should have separate [[billing]] but share a common backup vault. How would you set up cross-account access for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Instant Retrieval?

Correct Answer:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with necessary permissions.
2. Attach the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] vault.
3. In the destination account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user with required actions.
4. Configure trust relationships between the two accounts using AssumeRole.

Justification: By implementing this solution, departments can access the shared backup vault without exposing sensitive credentials or increasing overall costs.