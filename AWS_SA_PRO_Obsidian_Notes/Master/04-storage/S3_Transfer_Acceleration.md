**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration utilizes Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]'s globally distributed edge network to accelerate uploads to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets. It achieves this by providing a dedicated endpoint that uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]'s network to optimize data transit. The client's requests first go to the nearest [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] edge location, which then routes them to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. This process reduces latency, packet loss, and time spent on transiting the internet.

The service operates at the application layer (OSI Layer 7) and leverages Edge-optimized HTTP/HTTPS connections between clients and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] edge locations. Data is stored in an intermediate cache within these edge locations before being forwarded to the destination [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. This architecture allows for faster transfers, especially when transferring large files or high volumes of data over long distances.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service           | Use Case                                                              |
| ----------------- | -------------------------------------------------------------------- |
| [[Srinivas_Notes/S3|S3]] Transfer       | Large file uploads, real-time data ingestion, low-latency transfers   |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|S3 Replication]]    | Multi-region data duplication, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]                     |
| [[Srinivas_Notes/S3|S3]] Batch Copy     | Offline migration, bulk processing                                   |
| Direct Connect   | Dedicated connectivity between your network and AWS                    |
| [[Storage Gateway]] | Hybrid cloud storage, local [[api-gateway|caching]], backup, and archival               |

*Anti-pattern*: Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration for downloads. Instead, leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions for faster global retrieval.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy JSON Snippet:
```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": [
        "s3-transfer-acceleration:*"
    ],
    "Resource": "arn:aws:s3-transfer-acceleration:us-west-2:123456789012:expedited/*"
}
```
Cross-account access:
- Create a role in the source account with permissions to perform `s3:PutObject`, `s3:PutObjectAcl`, and `s3:ListBucket`.
- Attach a trust relationship policy to the role allowing the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity (user/role).
- In the destination account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user/role with permissions to assume the above role (via the `sts:AssumeRole` action).

Organization SCPs:
- Implement deny SCPs to prevent unauthorized creation of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration resources. Example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyS3TransferAccelerationCreate",
            "Effect": "Deny",
            "Action": [
                "s3-transfer-acceleration:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEquals": {
                    "aws:PrincipalOrgID": "o-123456789012"
                }
            }
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits:
- PUT, COPY, POST, or LIST operations per second: 100
- GET requests per second: 500

Exponential backoff strategies:
- Implement retry mechanisms using exponential backoff with jitter for handling throttled requests.

HA/DR patterns:
- Configure multiple [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration endpoints across different regions for geographical redundancy.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls:
- Enable/disable Transfer Acceleration per prefix or object tagging.
- Monitor usage via [[cloudwatch]] metrics, alarms, and [[Budgets]].

Calculation example:
- Uploading 1 GB from New York (US East) to Oregon (US West) costs $0.04 ($0.02 standard [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] PUT request + $0.02 Transfer Acceleration fee).

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A:
Your company has a centralized organization setup with several AWS accounts. They need to securely share data between their [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and a partner's [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket while maintaining compliance requirements. How would you implement a solution?

Correct answer:
1. Create a cross-account [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with required permissions in both the source and destination accounts.
2. Set up [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration for each bucket to improve data transfer performance.
3. Apply SCPs in the organization root account to restrict unauthorized changes to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration resources.

Incorrect answer:
1. Share the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket directly with the partner without proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Reasoning: Sharing the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket directly may not provide adequate [[appsync|security]] and governance control. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration, and SCPs ensure secure and efficient data sharing between the two parties.

Scenario B:
Your company needs to migrate 1 TB of data from its on-premises environment to AWS [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Which services should be used considering performance, reliability, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]?

Correct answer:
1. Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration to speed up data transfer.
2. Schedule data migrations during off-peak hours to minimize disruptions.
3. Periodically monitor data transfer costs and adjust migration schedules accordingly.

Incorrect answer:
1. Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Batch Copy to move data since they don't support transfer acceleration.

Reasoning: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration offers better performance and lower costs compared to other options like [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Batch Copy. However, it doesn't support automatic scheduling or monitoring features found in other tools such as [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]].