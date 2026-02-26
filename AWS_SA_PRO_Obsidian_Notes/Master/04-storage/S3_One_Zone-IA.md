## [[RDS_Instance_Types|1. Advanced Architecture]]: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA is a storage class that stores data in a single Availability Zone (AZ) at a lower cost than other [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]. It's designed for infrequently accessed data with a low probability of data loss due to an event impacting a single AZ.

### [[RDS_Instance_Types|Global Scale Considerations]]

One Zone-IA simplifies managing data across multiple accounts and regions by providing a centralized view of metadata using **[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]]**. This feature allows you to store specific versions of objects in a write-once-read-many (WORM) mode, ensuring data immutability. However, since data is stored in a single AZ, it may not be suitable for applications requiring multi-region redundancy.

### Under the Hood Mechanics

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA uses Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] server-side encryption (SSE) to protect data at rest using either SSE-S3 or SSE-KMS. Data can also be encrypted in transit using HTTPS. To optimize costs further, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering can automatically move data between One Zone-IA and other [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] based on usage patterns.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Factor | One Zone-IA | [[Srinivas_Notes/S3|S3]] Standard | [[Srinivas_Notes/S3|S3]] Intelligent-Tiering |
|---|---|---|---|
| Durability | Lower (99.5%) | Higher (99.999999999%) | High (99.999999999%) |
| Cost | Lowest | Highest | Medium |
| Suitability | Infrequent access, low probability of data loss | Highly available, mission-critical workloads | Unpredictable access patterns |

**Anti-patterns:**

* Storing mission-critical data without proper backup/recovery mechanisms.
* Using One Zone-IA for data requiring high durability or availability.

## [[RDS_Instance_Types|3. Security & Governance]]

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

To enforce fine-grained control over [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] resources, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy like the following example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::examplebucket/*"
            ],
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-storage-class": "ONEZONE-IA"
                }
            }
        }
    ]
}
```

### Cross-Account Access

Enable cross-account access with bucket [[policies]] allowing specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles from another account to perform actions on your One Zone-IA buckets.

### Organization Service Control [[policies]] (SCPs)

Use SCPs within [[organizations|AWS Organizations]] to restrict creating or modifying One Zone-IA buckets in member accounts. The following [[SCP]] example denies such actions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyOneZoneIA",
            "Effect": "Deny",
            "Action": [
              "s3:CreateBucket",
              "s3:PutBucketStorageClass"
            ],
            "Resource": "arn:aws:s3:::*",
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "s3:x-amz-storage-class": "ONEZONE-IA"
                },
                "Bool": {
                    "aws:ViaAWSService": "true"
                }
            }
        }
    ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits for One Zone-IA are similar to those of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard. If throttled, implement exponential backoff strategies to handle temporary [[api-gateway|errors]].

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] (HA/DR), consider replicating One Zone-IA data to another region using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]]. Alternatively, use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Cross-Region Replication (CRR) to create a mirror copy of One Zone-IA buckets in different regions.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include applying lifecycle management [[policies]] to transition data between One Zone-IA and other [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] based on age or access frequency. For instance, move data to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard after 30 days or when accessed.

Calculate One Zone-IA costs as follows:

* Estimate storage costs per month (pricing varies depending on location).
* Add retrieval costs if frequently accessing the data.
* Include any data transfer fees.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1: Multi-account Strategy

An organization has 3 AWS accounts storing media assets for various projects. They want to reduce costs while maintaining low retrieval times. Which strategy should they adopt?

*Correct answer:* Implement a multi-account [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] strategy using One Zone-IA for infrequently accessed data. Apply [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle Management [[policies]] to transition data between [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] based on project requirements.

*Incorrect answer:* Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard for all media assets in one account. *Explanation:* This approach would result in higher costs compared to using One Zone-IA.

### Scenario 2: Cross-Account Access

A company needs to grant a third-party vendor read-only access to their One Zone-IA bucket containing marketing materials. How should they set up permissions?

*Correct answer:* Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to access the One Zone-IA bucket. Share the role ARN with the third-party vendor, who can then assume the role and access the bucket.

*Incorrect answer:* Modify the bucket policy to allow public read access. *Explanation:* Public read access poses [[appsync|security]] risks, making it less desirable than using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.