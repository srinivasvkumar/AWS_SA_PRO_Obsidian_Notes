## [[RDS_Instance_Types|1. Advanced Architecture]]

[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] uses two services: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Cross-Region Replication (CRR) and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Same-Region Replication (SRR). CRR replicates objects across buckets in different regions, while SRR does so within the same region. Both services use replication rules that specify which objects to replicate and their destination buckets.

### [[RDS_Instance_Types|Global Scale Considerations]]

* Each source bucket can have up to one destination bucket per region.
* Object metadata is replicated verbatim, but data transfer objects may differ due to regional [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]].
* Update Visibility: Metadata updates are immediately visible, but content changes require a new replication request.
* Versioning must be enabled on both source and destination buckets.

### Under the Hood Mechanics

* Change Tracking: Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] determines object changes via event notifications or inventory lists.
* Eventual Consistency: Standard/Intelligent Tiering/One Zone-IA replicate immediately, [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]] have eventual consistency.
* Conflict Resolution: If an object has existing replication configurations, it won't be overwritten by another replication request.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Feature | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|S3 Replication]] | [[Srinivas_Notes/S3|S3]] Transfer Acceleration | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/11-migrations/datasync|AWS DataSync]] |
|---|---|---|---|
| Direction | Unidirectional | Unidirectional | Bi-directional |
| Protocol | HTTP(S) | UDP (TCP optional) | Various |
| Performance | Depends on network [[cloudformation|conditions]] | Faster than standard transfers | High throughput |
| Cost | Varies by region & size | Varies by region & size | Pay-per-use |
| Use Case | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster recovery]], compliance | Content delivery, remote offices | Large-scale migrations |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] when real-time synchronization is required, as there might be eventual consistency issues. Also, avoid using it for migrating large amounts of data, as other tools like [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] offer better performance and cost efficiency.

## [[RDS_Instance_Types|3. Security & Governance]]

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] control who can create/modify/delete replication rules. For example, you can allow users to add replication rules but prevent them from deleting existing ones:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutReplicationConfiguration",
                "s3:PutBucketReplicationConfiguration"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:RequestedOperation": [
                        "Delete"
                    ]
                }
            }
        }
    ]
}
```

### Cross-Account Access

To enable cross-account replication, configure a bucket policy on the source bucket allowing the destination account's principal to perform specific actions:

```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::DESTINATION_ACCOUNT_ID:root"
    },
    "Action": [
        "s3:GetObjectVersionForReposition",
        "s3:ListBucketVersions",
        "s3:ReplicateObject",
        "s3:ReplicateTags"
    ],
    "Resource": "arn:aws:s3:::SOURCE_BUCKET/*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVault": "SOURCE_VAULT"
        }
    }
}
```

### Organization SCPs

Service Control [[policies]] (SCPs) can limit [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] usage at an organizational level. For instance, deny creating replication rules outside specific regions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "s3:PutReplicationConfiguration",
                "s3:PutBucketReplicationConfiguration"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIgnoreCase": {
                    "aws:RequestedRegion": [
                        "us-west-2",
                        "eu-west-1"
                    ]
                }
            }
        }
    ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

### Throttling Limits

Maximum replication requests per second depend on the source bucket's region. For example, US East (N. Virginia) allows up to 500 PUT requests per second. If throttled, implement exponential backoff strategies to handle increased latencies and retries.

### HA/DR Patterns

Use [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] as part of a high availability strategy by configuring multiple replication destinations across various AWS Regions. This ensures data durability even if entire regions become unavailable.

## [[RDS_Instance_Types|5. Cost Optimization]]

### Granular Cost Controls

Control costs by specifying replication rules only for specific objects based on prefixes, tags, or object sizes. Monitor usage with Amazon [[cloudwatch|CloudWatch alarms]] and [[billing]] reports.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario A

You need to set up a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] solution for sensitive data stored in an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket named `source-bucket`. The [[dr]] site should reside in a different region, and all objects should be encrypted using AWS Key Management Service ([[kms]]). Which steps would you follow?

Correct Answer:

1. Enable versioning on both `source-bucket` and its destination bucket.
2. Create a [[kms]] key in the destination region.
3. Add a replication rule to `source-bucket` that includes the following settings:
   * Destination: Destination bucket in the other region.
   * Encryption Configuration: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS KMS]], selecting the [[kms]] key created in step 2.

Incorrect Answers:

- Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration instead of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] won't ensure data safety in case of a regional outage.
- Implementing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] lifecycle [[policies]] without enabling versioning could lead to accidental deletion of objects.

### Scenario B

As a Solutions Architect, you are tasked to implement a multi-account [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 replication]] strategy for a company with strict [[appsync|security]] requirements. The goal is to restrict the creation of replication rules to specific regions and accounts. How would you achieve this?

Correct Answer:

1. Set up an [[AWS Organization]] with a master account and member accounts.
2. Apply a Service Control Policy ([[SCP]]) to the organization that denies the creation of replication rules outside specified regions and accounts.

Incorrect Answers:

- Modifying [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] alone doesn't suffice because they don't span across multiple accounts.
- Creating separate Organizational Units (OUs) for each account isn't necessary since SCPs can be applied directly at the root level.