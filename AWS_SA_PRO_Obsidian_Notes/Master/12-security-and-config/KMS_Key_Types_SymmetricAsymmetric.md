**[[kms]] Key Types: Advanced Architecture**

[[kms]] offers two types of keys: symmetric and asymmetric. Symmetric keys are more common and used for encryption and decryption. They never leave [[kms]] unencrypted and are wrapped with an envelope encryption scheme using a data key generated outside of [[kms]]. Asymmetric keys consist of a public key for encryption and a private key for decryption. They are useful when you want to share encrypted data with [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]] who don't have access to your [[kms]] key.

[[RDS_Instance_Types|Global Scale Considerations]]:

* [[kms]] is a regional service and does not provide built-in global distribution. To achieve global scale, create custom solutions using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] or SDKs.
* For asymmetric keys, ensure low latency by creating replicas in multiple regions.

Under The Hood Mechanics:

* Symmetric keys can be directly managed through [[kms]] or imported from external sources.
* Asymmetric keys are created and managed solely within [[kms]].

**Comparison & Anti-Patterns**

| Service | Use Cases |
| --- | --- |
| [[kms]] | Data at rest, fine-grained crypto operations, SSE-KMS |
| [[Collab_Notes_detailed/Security/CloudHSM|CloudHSM]] | Highly regulated workloads, compliance requirements |
| AWS Managed AD / EDS | Directory services, [[api-gateway|authentication]] |
| Hardware [[appsync|Security]] Modules (HSMs) | Secrets management, certificate generation |

Anti-patterns:

* Using [[kms]] for transit encryption (use AWS WAF or [[ALB]] for SSL termination)
* Sharing [[kms]] keys across accounts without proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] permissions

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {"AWS": "*"},
            "Action": ["kms:GenerateDataKey*"],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "kms:ViaService": "ec2.us-west-2.amazonaws.com"
                },
                "Bool": {
                    "kms:CallerAccount": "123456789012"
                }
            }
        }
    ]
}
```

Cross-account access:

* Grant permissions to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles in different accounts
* Use [[organizations|AWS Organizations]] SCPs for centralized control

**Performance & Reliability**

Throttling Limits:

* Maximum requests per second: 10,000 (can request limit increase)
* Exponential backoff strategy: implement retry logic with Jitter

HA/DR Patterns:

* Create replica keys in other regions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]
* Enable cross-region replication for asymmetric key pairs

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:

* Implement tagging [[policies]] for better resource tracking
* Set up [[billing]] alarms based on usage thresholds

Calculation Examples:

* Symmetric key: $1/key/month + $0.10/10,000 requests
* Asymmetric key: $2.50/key/month + $0.15/10,000 requests

**Professional Exam Scenarios**

Scenario 1:
You need to implement a solution that allows you to securely store sensitive customer information while complying with strict regulatory requirements. This system should allow auditing and [[vpc-flow-logs|logging]] capabilities. Which solution would best fit these needs?

Correct Answer: [[kms]] with AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] integration
Justification: [[kms]] provides strong encryption and key management features. AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] logs all API calls made via [[kms]], enabling audit and compliance reporting.

Incorrect Answers:

* [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]]: While it meets compliance requirements, it lacks the ability to integrate with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]].
* SSE-KMS: It only supports encryption for Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] objects.

Scenario 2:
Your company manages a highly available web application that requires low latency and high performance encryption for user data stored in [[dynamodb]]. How would you optimize the encryption process to minimize latency and maximize performance?

Correct Answer: Use KMS-generated data keys combined with [[dynamodb]] encryption client-side
Justification: By performing encryption and decryption client-side, you reduce the number of [[kms]] requests and minimize latency. Additionally, you maintain full control over encryption and decryption processes.

Incorrect Answers:

* Encrypt data server-side using [[Lambda@Edge]]: Introduces additional latency due to server-side processing.
* Store data encrypted using SSE-DynamoDB: Not recommended as it doesn't support client-side encryption.