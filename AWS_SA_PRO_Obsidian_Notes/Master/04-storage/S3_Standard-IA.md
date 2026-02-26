## [[RDS_Instance_Types|1. Advanced Architecture]]
[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA (Standard Infrequent Access) is a storage class that provides a balance between availability and cost. Objects stored in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA have a minimum duration of 30 days, and after that period, they incur a retrieval fee. Under the hood, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA stores data across multiple geographically distinct AZs within a region, providing a durability point of 99.9%.

When it comes to [[RDS_Instance_Types|global scale considerations]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA supports intelligent-tiering, which automatically moves objects between performance tiers based on access patterns. This feature enables [[organizations]] to optimize costs while maintaining high availability. Moreover, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA can be used in conjunction with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration, enabling fast transfers of data over long distances through edge network locations.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]
The following table compares [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA with other [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]:

| Attribute | [[Srinivas_Notes/S3|S3]] Standard | [[Srinivas_Notes/S3|S3]] Intelligent-Tiering | [[Srinivas_Notes/S3|S3]] One Zone-IA | [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] |
|---|---|---|---|---|
| Minimum storage duration | None | None | None | 90 days |
| Retrieval fee | None | None | None | Yes |
| Durability | 99.99% (single AZ) | 99.99% (single AZ) | 99.99% (single AZ) | 99.999999999% (11 nines) |
| Use case | Frequently accessed data | Automatically moving data between tiers | Single AZ backup and archival data | Long term retention and archive |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA when data is frequently accessed or has a minimum storage duration shorter than 30 days. In such cases, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering would be more appropriate.

## [[RDS_Instance_Types|3. Security & Governance]]
To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], you can use JSON snippets like the following example, allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to perform specific actions in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA:
```json
{
    "Effect": "Allow",
    "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
    ],
    "Resource": "arn:aws:s3:::example-bucket/*",
    "Condition": {
        "StringEqualsIgnoreCase": {
            "s3:x-amz-storage-class": "STANDARD_IA"
        }
    }
}
```
Cross-account access can be achieved by configuring bucket [[policies]] that allow specific principal types from another account to perform certain actions. Additionally, Service Control [[policies]] (SCPs) at the organization level can enforce restrictions on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] usage, ensuring consistent [[appsync|security]] postures across accounts.

## [[RDS_Instance_Types|4. Performance & Reliability]]
[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA imposes several throttling limits, including 3,500 PUT/POST/COPY requests per second and 5,500 GET/SELECT requests per second. To manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], exponential backoff strategies should be implemented. For example, if a request fails due to throttling, wait for twice the amount of time before retrying the request.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA involve replicating data across regions using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] or storing data in multiple AZs within a single region. This ensures data resiliency in the event of regional outages or disasters.

## [[RDS_Instance_Types|5. Cost Optimization]]
Granular cost controls for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA can be achieved by setting lifecycle management [[policies]] that transition objects to lower-cost [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] or delete them entirely after a specified period. Calculating costs involves understanding the total volume of data stored as well as the number of read/write operations performed.

For instance, let's consider a scenario where you store 1 TB of data in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA for 45 days, resulting in $0.011 per GB-month storage cost. The retrieval fee is $0.01 per GB. If you perform 100,000 GET requests during this period, each costing $0.0004, your total cost will be:

Storage: 1024 GB \* $0.011/GB/month \* 1 month = $11.26
Retrieval: 1024 GB \* $0.01/GB = $10.24
GET Requests: 100,000 \* $0.0004 = $40
Total: $11.26 + $10.24 + $0.40 = $21.90

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]
### Scenario 1
An e-commerce company needs to store product images that are infrequently accessed but require low latency. They expect to store around 500 GB of data initially and add up to 100 GB monthly. Which [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] storage class should they choose?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA offers a good balance between availability and cost for infrequently accessed data. It ensures low latency for initial access while minimizing costs compared to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard.

Incorrect Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA does not meet their requirements since it only stores data in a single AZ without redundancy. [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] is also unsuitable because it has a minimum storage duration of 90 days.

### Scenario 2
A media company wants to ensure secure cross-account access to their [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA content library for their development team in another AWS account. How should they grant access?

Correct Answer: Implement a bucket policy that allows the specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles from the development account to perform desired actions on the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-IA bucket.

Incorrect Answer: Using SCPs to enforce permissions is incorrect because SCPs work at the organization level, not at the individual bucket level. Similarly, creating a custom [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] permission policy directly attached to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user in the development account is insufficient, as it doesn't address the cross-account aspect.