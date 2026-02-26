## Advanced Architecture
At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive is a storage class optimized for long-term retention of infrequently accessed data. It offers 99.999999999% durability and is designed to be exceptionally economical.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive stores objects in a "vault" which can hold an unlimited number of archives. Each archive may contain up to 40TB of data. The system uses advanced techniques such as data redundancy across multiple geographic locations and erasure coding to ensure data resiliency while minimizing costs.

[[RDS_Instance_Types|Global scale considerations]] include the fact that retrieval times for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive are significantly longer than other Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] due to its focus on extreme low cost. A single retrieval request typically takes 12 hours to complete. If immediate access to your data is required, you should consider using another storage class like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA instead.

## Comparison & Anti-Patterns

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive | Regulatory compliance, digital preservation, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]           | Frequent retrievals needed within 1-3 months                |
| [[Srinivas_Notes/S3|S3]] One Zone-IA       | Long-term storage, but more frequent access than Deep Archive    |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive when frequent retrievals are necessary. This will lead to unnecessary delays and increased costs due to retrieval fees.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive might involve permissions for listing vaults, creating archives, deleting archives, and initiating restores. Here's a JSON snippet example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListVault",
                "s3:GetVaultNotifications",
                "s3:PutVaultNotifications"
            ],
            "Resource": "arn:aws:s3:::example-deep-archive-bucket/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::example-deep-archive-bucket/*",
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "s3:x-amz-storage-class": "GLACIER_DEEP_ARCHIVE"
                }
            }
        }
    ]
}
```

Cross-account access can be achieved by adding a principal from another account to your [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy. For instance:

```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": [
        "s3:PutObject",
        "s3:GetObject"
    ],
    "Resource": "arn:aws:s3:::example-deep-archive-bucket/*",
    "Condition": {
        "StringEquals": {
            "s3:x-amz-acl": "authenticated-read"
        }
    }
}
```

Organization SCPs (Service Control [[policies]]) can enforce usage of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive across accounts within an organization. However, they cannot directly restrict storage class selection.

## Performance & Reliability
Throttling limits for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive are quite high compared to other [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]. You can make up to 300 requests per second for initiating a restore operation. To avoid exceeding these limits, implement exponential backoff strategies during periods of heavy traffic.

High availability/disaster recovery patterns could involve storing critical data in different regions using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]], then transitioning replicated copies to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be implemented through metrics and alerts based on API calls, bytes downloaded, or number of archives restored. [[billing]] reports can also help monitor spending trends.

Calculation Example:
Assuming you store 1 petabyte (PB) of data in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive for one year, your annual storage cost would be approximately $300 ($0.00099 per GB per month * 1024 TB * 12 months). If you retrieve 10% of this data once a year, your annual retrieval cost would be around $1500 (assuming $0.01 per GB retrieved).

## Professional Exam Scenarios

Scenario 1:
Your company must comply with regulatory requirements mandating 7-year storage of financial records. Due to limited budget, you want to minimize costs. Which storage solution should you choose?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive.
Justification: It provides the lowest cost for long-term storage and meets the 7-year retention requirement.

Incorrect Answers:
- [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard: Too expensive for long-term storage without frequent access.
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]]: Not suitable for storing archival data.

Scenario 2:
An e-commerce application needs to store order information for 3 years while meeting strict performance requirements. What's the best strategy?

Correct Answer: Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering.
Justification: Both provide high throughput and low latencies, ensuring good performance. They also offer features like cross-region replication and versioning.

Incorrect Answers:
- [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive: Inappropriate due to slow retrieval times.
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]: Primarily designed for use with [[ec2]] instances, not optimal for object storage.