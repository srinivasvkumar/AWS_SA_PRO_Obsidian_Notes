## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering is a sophisticated storage management service that optimizes data storage costs by automatically moving objects between four access tiers: Frequent Access (FA), Infrequent Access (IA), Archive Instant Access (AIA), and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] (GLAC) based on their usage patterns. It uses machine learning algorithms to analyze object access patterns and determine the most cost-effective tier for each object.

The service operates at the bucket level and supports both standard and intelligent-tiering buckets within an account. The underlying architecture consists of multiple layers of storage (FA, IA, AIA, and GLAC) and a metadata layer responsible for tracking object access patterns and migrations.

[[RDS_Instance_Types|Global Scale Considerations]]:
Intelligent-Tiering supports cross-region replication, enabling you to create a primary bucket in one region and replicate it to secondary buckets in other regions. This feature allows you to maintain low-latency access to your data while ensuring data durability and resiliency across geographically dispersed locations.

Under the Hood Mechanics:
Intelligent-Tiering monitors object access patterns and migrates them to the appropriate tier based on the following rules:

* If an object has not been accessed for 30 consecutive days, it will be moved from FA to IA.
* If an object has not been accessed for 60 consecutive days after being moved to IA, it will be moved from IA to AIA or GLAC.
* Objects in AIA can be restored to IA within seconds, while objects in GLAC can take minutes to hours depending on the retrieval option chosen.

## Comparison & Anti-Patterns

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[Srinivas_Notes/S3|S3]] Intelligent-Tiering | Data with unpredictable access patterns, frequently accessed but still cost-sensitive |
| [[Srinivas_Notes/S3|S3]] Standard     | Highly performant workloads requiring frequent access                |
| [[Srinivas_Notes/S3|S3]] One Zone-IA   | Data requiring infrequent access with a lower cost than [[Srinivas_Notes/S3|S3]] IA         |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]      | Long-term archival storage with minimal access requirements            |

Anti-Patterns:

* Using Intelligent-Tiering for data with predictable access patterns may lead to unnecessary migrations and increased costs.
* Storing small objects (less than 128KB) in Intelligent-Tiering as migration costs may outweigh the benefits.

## [[appsync|Security]] & Governance

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
To grant users access to specific tiers, you can use the `s3:PutObject` action with the `StorageClassAnalysis` parameter set to the desired tier. For example:
```json
{
    "Effect": "Allow",
    "Action": ["s3:PutObject"],
    "Resource": ["arn:aws:s3:::example-bucket/*"],
    "Condition": {
        "StringEquals": {
            "s3:x-amz-storage-class-analysis": ["ARCHIVE"]
        }
    }
}
```
Cross-Account Access:
To enable cross-account access, you must configure a bucket policy that grants permissions to the source and destination accounts. Here's an example policy allowing a user in Account B to read from a bucket in Account A:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/user-in-account-b"
            },
            "Action": ["s3:GetObject"],
            "Resource": "arn:aws:s3:::example-bucket/*"
        }
    ]
}
```
Organization SCPs:
You can enforce organizational [[policies]] using Service Control [[policies]] (SCPs) at the organization level. For instance, you can restrict the creation of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets to specific [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": ["s3:CreateBucket"],
            "Resource": ["arn:aws:s3:::*"],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "s3:storage_class": ["INTELLIGENT_TIERING", "ONEZONE_IA", "GLACIER", "DEEP_ARCHIVE"]
                }
            }
        }
    ]
}
```

## Performance & Reliability

Throttling Limits:
Intelligent-Tiering supports a maximum of 3,500 PUT/COPY/POST/DELETE requests per second for a single bucket. To handle higher request rates, you can implement exponential backoff strategies to ensure availability and reliability during periods of high demand.

HA/DR Patterns:
Intelligent-Tiering supports cross-region replication, which enables you to create a highly available and durable solution. By configuring a primary bucket and secondary buckets in different regions, you can maintain low-latency access to your data while ensuring data durability and resiliency across geographically dispersed locations.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular Cost Controls:
Intelligent-Tiering provides granular cost controls by monitoring object access patterns and automatically migrating them between tiers based on their usage. However, it's essential to [[nonportable_instructions|review]] and optimize your access patterns further to minimize costs. For example, implementing lifecycle [[policies]] to transition older objects to cheaper [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] such as One Zone-IA or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] can help reduce overall costs.

Calculation Examples:
Consider a bucket containing 1 TB of data stored in Intelligent-Tiering with the following access pattern:

* 10% of the data is accessed daily
* 40% of the data is accessed once every 30 days
* 50% of the data is accessed once every 90 days

Based on these assumptions, you can calculate the expected monthly cost as follows:

* First 5 GB (10% of 50 GB): $0.023 per GB = $0.115
* Next 200 GB (40% of 500 GB): $0.0125 per GB = $2.50
* Remaining 750 GB (50% of 1.5 TB): $0.01 per GB = $7.50

Total Monthly Cost: $10.115


## Professional Exam Scenarios

Scenario 1:
Your organization has a 10 TB [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket containing scientific research data. The data has varying access patterns, with some files being accessed daily, [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]] weekly, and the rest rarely. Design a cost-optimized solution using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering.

Correct Answer:
Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering to store the entire 10 TB bucket. Implement lifecycle [[policies]] to move older data to cheaper [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] like One Zone-IA or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]].

Incorrect Answer:
Store the entire dataset in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard due to varying access patterns.

Justification:
Storing all the data in Intelligent-Tiering would allow automatic migration between tiers based on access frequency, reducing overall costs compared to storing the entire dataset in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard. Implementing lifecycle [[policies]] ensures that less frequently accessed data moves to cheaper [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]], further minimizing costs.

Scenario 2:
Your company needs to store sensitive financial information in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], adhering to strict compliance regulations. The data must remain encrypted and accessible only by specific individuals. Which [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] features should you use?

Correct Answer:
Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering along with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Server-Side Encryption (SSE) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to control access.

Incorrect Answer:
Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard without encryption or access control mechanisms.

Justification:
Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering allows for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] without compromising [[appsync|security]]. Enabling SSE ensures that data remains encrypted at rest, while [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs provide fine-grained access control, ensuring that only authorized individuals can access the sensitive financial information.