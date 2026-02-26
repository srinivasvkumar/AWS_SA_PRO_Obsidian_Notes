## [[RDS_Instance_Types|1. Advanced Architecture]]
[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]] are a feature of Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] that simplifies managing application data access at scale. An [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Access Point is an isolated view into the bucket, including its own permissions, metrics, and [[vpc-flow-logs|logging]] configurations. It enables you to manage data access at a very granular level.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]] utilize a unique combination of the following elements:

- **Object namespace:** Each Access Point has a dedicated object namespace within its parent bucket. Object key names must be unique within each Access Point but can overlap across different Access Points in the same bucket.
- **Access Point policy:** JSON document scoped to the Access Point that overrides [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket-level [[policies]]. It allows or denies operations based on user identity, resource type, and condition context.
- **[[vpc-flow-logs|Logging]] configuration:** Defines where to store logs for API calls made through the Access Point.

At a global scale, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]] support multiple regions by replicating objects using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]], enabling low-latency reads while keeping the master copy in a single region. This setup provides a unified namespace across regions without increasing the operational burden.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Feature                         | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|S3 Access Points]]          | Alternatives           |
| ------------------------------- | ------------------------- | --------------------- |
| Namespace isolation            | Yes                       | [[Srinivas_Notes/S3|S3]] Bucket [[policies]]    |
| User-based permissions          | Yes                       | Bucket [[policies]]      |
| [[vpc-flow-logs|Logging]]                        | Yes                       | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]            |
| Global Scale                   | Limited                   | [[Git_hub_notes/certified-aws-solutions-architect-professional-main/04-storage/s3|S3 Replication]]        |
| Performance optimization       | Read-only replicas         | None                 |
| Cost reduction                 | Granular control           | None                 |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]] as a replacement for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Bucket Policy when there's no need for separate namespaces or access rules. Also, it's not recommended to use Access Points when the primary goal is performance optimization since other services like Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] or [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] might provide better results.

## [[RDS_Instance_Types|3. Security & Governance]]
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve specifying various [[cloudformation|conditions]] such as AWS principal ARN, AWS action, and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] object names. Here's an example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": ["arn:aws:iam::123456789012:user/Alice"]},
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:us-west-2:111122223333:accesspoint/MyAccessPoint",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "public-read"
        }
      }
    }
  ]
}
```
Cross-account access is possible by adding external account numbers in the `Principal` field, while Organization Service Control [[policies]] (SCPs) can enforce restrictions on creating/modifying Access Points.

## [[RDS_Instance_Types|4. Performance & Reliability]]
Throttling limits vary depending on the operation mode (standard, Intelligent-Tiering, One Zone-IA, etc.). For standard [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]], the limit is 3,500 PUT/COPY/POST/DELETE requests per second. Exponential backoff strategies should be implemented for handling throttle [[api-gateway|errors]].

HA/DR patterns rely on [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]], which supports cross-region replication between Access Points. In addition, read-only replicas can offload traffic from the source Access Point, improving overall system performance.

## [[RDS_Instance_Types|5. Cost Optimization]]
Granular cost controls come from setting up individual Access Points per application or team, allowing independent monitoring and management. The following formula calculates the cost of storing objects via Access Points:

`(Number of objects * Size of objects * Storage class rate) + (Number of requests * Request rates)`

For instance, if you have 1000 objects stored in the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard tier ($0.023/GB/month) accessed 100 times daily ($0.005 per 1000 PUT requests):

`(1000 * 0.001 GB * $0.023/GB/month) + (100 * $0.005/1000) = ~$0.0233/month`

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]
### Scenario 1:
A company wants to grant specific users read-only access to their [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets while maintaining separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|permission sets]]. How would you implement this using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]]?

Correct Answer: Create an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Access Point per user with a custom policy granting read-only access.

Incorrect Answer: Implement these permissions directly at the bucket level.

Justification: Managing separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|permission sets]] requires isolating them within Access Points. Applying user-specific [[policies]] directly at the bucket level could lead to confusion and potential [[appsync|security]] risks.

### Scenario 2:
An organization needs to minimize latency when serving static assets across multiple geographic locations while ensuring high availability and [[dr]] capabilities. Which tools should they use alongside [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]]?

Correct Answer: Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]].

Incorrect Answer: Using only [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Access Points]] or relying on [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]].

Justification: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] caches frequently requested content closer to end-users, reducing latency. Simultaneously, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Replication]] ensures data redundancy and availability during failures. Global Accelerator may improve connectivity but doesn't address the core requirements of low latency and high availability.