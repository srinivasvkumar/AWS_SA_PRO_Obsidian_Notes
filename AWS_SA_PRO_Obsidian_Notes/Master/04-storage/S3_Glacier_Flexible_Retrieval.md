## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval is a storage class that provides a balance between access time and cost for data retrieval. It offers three retrieval options: expedited (minutes), bulk (hours), and standard (days). The service uses a combination of cold storage and a separate index system for object metadata. This design allows for faster data retrieval compared to other archival services while maintaining cost-effectiveness.

Global scale is achieved through a network of regional "vaults" responsible for storing customers' data. Each vault has an associated index system allowing users to interact with objects without needing to know their physical location. To ensure low latency and high availability, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval employs multiple index systems within each region, distributing data across them using consistent hashing algorithms.

When you create a bucket in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], it automatically selects the most appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval vault based on [[policies]] like data residency and business continuity requirements. Objects stored in the bucket are then encrypted, split into chunks, and distributed across multiple index systems in the selected vault.

## Comparison & Anti-Patterns

| Service           | Ideal Use Case                                                              |
|------------------|----------------------------------------------------------------------------|
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flex. | Long-term archival data with occasional retrieval needs (minutes to days) |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep  | Rare long-term archival data (months to years)                               |
| [[Srinivas_Notes/S3|S3]] Intelligent   | Dynamic workloads requiring automatic tiering based on access frequency    |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval for frequently accessed data or as a primary storage option.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created using JSON statements specifying various permissions. Here's an example granting read-only access to all [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval buckets in an account:

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
                    "glacier:StorageClass": "GLACIER_FLEXIBLE_RETENTION"
                }
            }
        }
    ]
}
```

Cross-account access can be established by adding the ARN of the external account to your [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy. For organization-wide restrictions, Service Control [[policies]] (SCPs) can enforce specific [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] at the organizational level.

## Performance & Reliability

Throttling limits exist for API calls related to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval operations. If these limits are exceeded, exponential backoff strategies should be implemented to avoid overwhelming the system.

HA/DR patterns involve creating replicas of critical data across different regions or accounts. In case of failure, requests can be redirected to secondary locations.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include setting lifecycle management [[policies]] to transition objects between different [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] based on age or usage patterns. Calculation examples:

- Assume 100TB of data stored in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval with a monthly cost of $100 (assuming $0.01/GB).
- A lifecycle management policy transitions objects to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard after 90 days, resulting in additional storage costs but reducing retrieval fees.

## Professional Exam Scenarios

Scenario 1: An e-commerce company wants to store order details and customer information for legal reasons. They need quick access to this data once a year during audits but do not want to pay high retrieval costs. What storage solution would you recommend?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval. This service provides low-cost storage with flexible retrieval times, making it ideal for infrequently accessed data.

Incorrect Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard. This storage class is more expensive than [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval and does not provide cost savings for infrequently accessed data.

Scenario 2: A media company stores large video files in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Flexible Retrieval. Due to recent growth, they experience throttling issues when listing jobs. How can they improve performance without increasing costs?

Correct Answer: Implement exponential backoff strategies. By introducing delays between consecutive API calls, the application can avoid overwhelming the system.

Incorrect Answer: Increase the number of parallel retrieval requests. While this may temporarily solve the issue, it increases costs due to higher API call frequencies and potential overuse of resources.