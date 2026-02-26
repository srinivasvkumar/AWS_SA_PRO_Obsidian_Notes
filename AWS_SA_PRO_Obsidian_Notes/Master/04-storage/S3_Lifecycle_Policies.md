**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]] enable automatic transition and expiration of objects across different [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]], optimizing cost and performance. They operate at the bucket level, applying rules to object prefixes or whole buckets. An advanced pattern is using multiple [[policies]] per bucket, managing unique lifecycles.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] uses object versioning and metadata (creation date, standard/intelligent tiering) to enforce lifecycle actions. Transition actions move objects between classes, while expiration removes them entirely. Global [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] operates in 24 regions with real-time replication via [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration and cross-region replication.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Cases |
| --- | --- |
| [[Srinivas_Notes/S3|S3]] Lifecycle [[policies]] | Objects with predictable access patterns, archival, long-term retention. |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Vault LTSP | Versioned objects requiring [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|legal hold]], lower retrieval costs than [[Srinivas_Notes/S3|S3]]. |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|S3 Object Lock]] | Compliance-driven immutability, WORM (Write Once Read Many) scenarios. |

Anti-pattern: Using Lifecycle [[policies]] for short-term object retention without monitoring. This can result in accidental deletions and increased egress costs during restoration.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] control who can create, delete, and modify lifecycle [[policies]]. Example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutLifecycleConfiguration"
            ],
            "Resource": "arn:aws:s3:::examplebucket/*"
        }
    ]
}
```

Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and bucket [[policies]]. For Organization SCPs, enforce consistent tagging or block public bucket [[policies]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyPublicReadWriteMKLAC",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::examplebucket/*",
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]] have no throttling limits but may experience delays due to eventual consistency. To handle this, implement an exponential backoff strategy when checking rule execution status.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute data globally using cross-region replication, enabling versioning and lifecycle [[policies]] in each region.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include setting individual storage class costs based on usage patterns, applying lifecycle [[policies]] per object prefix, and leveraging metrics like age and size. Calculating costs involves understanding storage class pricing, retrieval fees, and transfer costs.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

*Scenario 1:* A company has two accounts, Prod and Backup. The Prod account contains sensitive data that must be transferred to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] after 90 days. Design a solution using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], and cross-account access.

Correct answer: Implement a lifecycle policy in the Prod account with a transition action moving objects to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] after 90 days. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the Prod account allowing the Backup account to read objects. Update the Backup account's bucket policy to permit listing and getting objects from the Prod account.

Incorrect answer: Configuring a single bucket policy in the Prod account granting cross-account access to Backup. This doesn't address the requirement for a lifecycle policy.

*Scenario 2:* A media organization stores petabytes of video content in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard. They want to minimize costs by automatically transitioning older files to cheaper storage tiers. Describe how to achieve this using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent Tiering and Lifecycle [[policies]].

Correct answer: Enable [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent Tiering for the videos bucket, which will monitor access patterns and transition infrequently accessed objects to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA or One Zone-IA. Set up a lifecycle policy with an expiration action to remove objects from the bucket after a specific period (e.g., 1 year) if they haven't been accessed.

Incorrect answer: Applying only a lifecycle policy to transition objects based on their creation date without considering access patterns might not minimize costs effectively.