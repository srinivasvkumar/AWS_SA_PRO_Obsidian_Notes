## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] is a feature that allows you to store a subset of objects in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets in a "immutable" state, preventing them from being deleted or overwritten for a specified [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. This is achieved by using a combination of [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket versioning and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 object lock]] settings.

The following diagram illustrates the internal workings of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]]:
```mermaid
graph LR
    A[S3 Bucket] --> B[Versioning Enabled]
    B --> C[Object Lock Settings Configured]
    C --> D[Objects Stored With Object Lock]
    D --> E[Legal Hold Applied]
    E --> F[Immutable State]
```
When an object is stored in a bucket with object lock enabled, it can be put into an immutable state by applying a [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|legal hold]]. The [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|legal hold]] can be either a governance mode or a compliance mode. In governance mode, the object cannot be deleted or overwritten for the duration of the [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]]. In compliance mode, the object cannot be deleted or overwritten until the minimum [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] has been met.

On a global scale, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] operates seamlessly across all regions. However, there are some [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] to be aware of. For example, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] is not supported in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration, and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] requests must be made via the REST API or the AWS SDKs.

## Comparison & Anti-Patterns

[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] should not be used in the following scenarios:

| Scenario | Alternative Solution |
| --- | --- |
| Data that does not require immutability | Use [[Srinivas_Notes/S3|S3]] with versioning and lifecycle [[policies]] |
| Large number of objects requiring immutability | Consider using [[Srinivas_Notes/S3|S3]] [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Vault Lock |
| Real-time data processing | Use [[Srinivas_Notes/S3|S3]] Intelligent-Tiering or [[Srinivas_Notes/S3|S3]] Standard-IA instead |

Common anti-patterns include:

* Using [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] as a replacement for proper backup and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] strategies
* Applying Object Lock at the account level rather than at the individual bucket level
* Not configuring appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and permissions for [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]]

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] might look like this:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectLegalHold",
                "s3:PutObjectRetention"
            ],
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
                "StringEqualsIfExists": {
                    "s3:x-amz-object-lock-legal-hold": [
                        "ON"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:DeleteObjectVersion",
                "s3:DeleteObjectVersionTagging"
            ],
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
                "Null": {
                    "s3:x-amz-object-lock-retain-until-date": true,
                    "s3:x-amz-object-lock-legal-hold-status": false
                }
            }
        }
    ]
}
```
Cross-account access can be granted using bucket [[policies]], while Organization SCPs can enforce [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] at the organization level.

## Performance & Reliability

There are no throttling limits specific to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]], but general [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] throttling limits apply. Exponential backoff strategies should be employed when making repeated [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] supports cross-region replication and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration, but note that [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] is not supported in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Accceleration.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] can be implemented by enabling versioning and using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]] to transition older versions to lower-cost [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]. Calculating costs requires understanding the additional charges associated with [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]]:

* Additional storage requirements for object lock metadata
* Increased PUT and SELECT request costs due to versioning
* Additional data transfer costs for retrieving previous versions

## Professional Exam Scenarios

Scenario 1: A company wants to implement [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] on their existing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets to ensure regulatory compliance. They have a large number of objects and need to minimize costs. What solution would you recommend?

Correct Answer: Enable [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] on a per-bucket basis and use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Lifecycle [[policies]] to transition older objects to lower-cost [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]].

Incorrect Answer: Enable [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] at the account level and use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Vault Lock for archival purposes.

Justification: Enabling [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] at the account level is too broad and may result in unnecessary costs. [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]] Vault Lock is intended for long-term archival and retrieval, which is not required in this scenario.

Scenario 2: An organization wants to restrict [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] usage to specific users within their AWS account. How would you implement this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]?

Correct Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that allows PutObject, PutObjectLegalHold, and PutObjectRetention actions only if the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|s3]]:x-amz-object-lock-legal-hold header is set to ON.

Incorrect Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that allows PutObject, PutObjectLegalHold, and PutObjectRetention actions without any [[cloudformation|conditions]].

Justification: Allowing PutObject, PutObjectLegalHold, and PutObjectRetention actions without any [[cloudformation|conditions]] grants unlimited access to these operations, which may not be desired. The correct answer provides more granular control over which objects can be locked.