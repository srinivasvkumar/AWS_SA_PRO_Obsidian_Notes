## [[RDS_Instance_Types|1. Advanced Architecture]]
[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard is a storage class that delivers high durability, availability, and performance. It replicates data across multiple facilities within a region and stores it in multiple Availability Zones (AZs) for durability. [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard provides 99.999999999% durability and 99.99% availability.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard uses a concept called "buckets" to store objects (files). Objects consist of data and metadata, including system metadata like version ID, HTTP headers, and optional user-defined metadata. Each object has a unique key (path) within its bucket.

Objects are distributed across multiple devices and facilities using a technique called "object partitioning." This ensures scalability and high performance when dealing with large datasets.

At the regional level, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard supports cross-region replication, enabling automatic real-time replication of new objects to another region. This feature helps meet compliance requirements, build multi-region applications for lower latency, or create secondary copies for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service                 | Use Case                                                              |
| ----------------------- | -------------------------------------------------------------------- |
| [[Srinivas_Notes/S3|S3]] Intelligent-Tiering | Ideal for workloads with unpredictable access patterns and frequent changes between access tiers. |
| [[Srinivas_Notes/S3|S3]] One Zone-IA          | Suitable for non-critical data with infrequent access and lower durability requirements. |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]             | Long-term archival storage with retrieval times from minutes to hours. |
| [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive | Extreme low-cost long-term archive, retrieval time up to 12 hours.    |

Anti-patterns include storing infrequently accessed data on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard instead of more cost-effective options like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]], or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive. Another anti-pattern is storing critical data on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard without proper replication and backup strategies.

## [[RDS_Instance_Types|3. Security & Governance]]
Implementing least privilege access control is essential for securing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] resources. Here's an example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy restricting access to specific buckets:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket-a/*",
                "arn:aws:s3:::my-bucket-b/*"
            ]
        }
    ]
}
```

Cross-account access can be established through Bucket [[policies]] granting permissions to external AWS accounts. For centralized management, [[organizations]] Service Control [[policies]] (SCPs) can enforce restrictions at the organizational level.

## [[RDS_Instance_Types|4. Performance & Reliability]]
Throttling limits in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] depend on the API call type. If throttled, implement exponential backoff strategies to handle increased waiting periods before retrying.

For High Availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] ([[dr]]), [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard supports cross-region replication as mentioned earlier. In addition, customers can leverage [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration to speed up uploads and downloads by routing traffic through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] edge locations.

## [[RDS_Instance_Types|5. Cost Optimization]]
Granular cost controls in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] involve monitoring usage and applying various techniques such as:

* Implementing lifecycle [[policies]] to transition older objects to cheaper [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]].
* Enforcing bucket-level quotas to prevent unexpected charges.
* Using metrics and alerts to monitor costs proactively.
* Storing large files in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] using server-side encryption (SSE-S3) instead of client-side encryption to save bandwidth costs.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario A
You are tasked with designing a solution for a customer who needs to store large amounts of image data with varying access frequencies. The customer requires 99.9% availability and expects to retrieve images within milliseconds. Which storage strategy would you recommend?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard offers the required availability and performance characteristics. While other [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] might provide similar performance, they lack the desired availability guarantees.

Incorrect Answers:

* [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering could result in higher costs due to frequent transitions between access tiers.
* [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] One Zone-IA does not offer the required availability guarantee.
* [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Deep Archive have longer retrieval times than needed.

### Scenario B
A media company wants to ensure secure access to their [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets containing sensitive video content. They want to limit object deletion capabilities while retaining full read/write permissions. What combination of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] features and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should be used to achieve this goal?

Correct Answer:

* Enable [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Versioning to maintain previous versions of objects.
* Implement a delete-object lifecycle policy that moves deleted objects to a specific [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket owned by the same account.
* Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants [[AWS_SA_PRO_Obsidian_Notes/Master/S3|s3]]:DeleteObjectVersion only to authorized users or roles.

Incorrect Answers:

* Deleting the bucket itself does not address the issue since the media company still requires read/write access to the bucket.
* Modifying bucket-level permissions without implementing object versioning and granular delete-object [[policies]] will not sufficiently protect the content.