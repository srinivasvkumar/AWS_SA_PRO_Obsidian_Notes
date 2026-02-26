**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] stores, rotates, and retrieves database credentials, API keys, and other secrets. It has a global infrastructure that supports multiple regions and availability zones. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS KMS]] for encryption at rest and AWS Key Management Service ([[kms]]) for encryption in transit using TLS 1.2.

Internally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] uses the [[lambda|AWS Lambda]] service to perform secret rotation tasks. The [[lambda]] function is responsible for connecting to the target resource and updating its credentials. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] also provides automatic rotation of secrets stored in [[parameter-store|AWS Systems Manager Parameter Store]].

[[RDS_Instance_Types|Global scale considerations]] include replicating secrets across regions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes. This can be achieved by creating a new version of the secret in the destination region or by copying the secret using the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] API.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] | Frequent secret rotation, integration with AWS services, and centralized management. |
| [[parameter-store|AWS Systems Manager Parameter Store]] | Infrequent secret rotation, standalone use, and compatibility with non-AWS services. |
| Hardcoded Credentials | Not recommended due to [[appsync|security]] risks and lack of automation capabilities. |

Common anti-patterns include using hardcoded credentials instead of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]], storing sensitive data in plaintext, and not enabling automatic secret rotation.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] include allowing specific users or roles to create, update, delete, and retrieve secrets. Here's an example JSON policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue",
                "secretsmanager:DescribeSecret"
            ],
            "Resource": [
                "arn:aws:secretsmanager:us-east-1:123456789012:secret:MyDatabaseSecret-a1b2c3"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:PutSecretValue",
                "secretsmanager:UpdateSecret",
                "secretsmanager:TagResource",
                "secretsmanager:DeleteResource"
            ],
            "Resource": [
                "arn:aws:secretsmanager:us-east-1:123456789012:secret:MyDatabaseSecret-a1b2c3"
            ],
            "Condition": {
                "StringEqualsIfExists": {
                    "aws:SourceVpce": "vpce-1234abcd5678efgh"
                }
            }
        }
    ]
}
```

Cross-account access involves granting permissions to resources in another account. This can be done using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or by specifying the source account in a resource-based policy.

Organization Service Control [[policies]] (SCPs) allow you to enforce [[appsync|security]] [[policies]] across all accounts within an organization. For example, an [[SCP]] can deny specific actions related to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] or restrict which users can manage secrets.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] depend on the region and can be found in the AWS documentation. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies such as increasing the wait time between retries and limiting the number of retries.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve replicating secrets across regions and ensuring that the underlying AWS resources support HA and [[dr]] requirements. Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover mechanisms to automatically switch to standby resources during failures.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include monitoring usage through [[cloudwatch]] metrics, setting alarms for unusual activity, and configuring [[billing]] alerts for specific costs. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] charges based on the number of secrets managed, the frequency of API requests, and the amount of data transferred.

Here's an example calculation for monthly costs:

* Number of secrets: 10
* Secret size: 1 KB
* API calls per month: 100,000
* Data transfer out per month: 1 GB

Monthly cost = (Number of secrets \* Price per secret + API calls \* Price per API call + Data transfer \* Price per GB)

= ($0.40 \* 10 + $0.05 \* 100,000 + $0.12 \* 1)

= $4,050.12

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Question 1: A company needs to store and manage secrets for their production databases. They want to enable automatic rotation every 30 days and ensure that only authorized users have access to these secrets. Which AWS services should they use?

Correct Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Explanation: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] allows for automatic rotation of secrets and integration with AWS services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be used to control who has access to these secrets.

Incorrect Answer: Systems Manager Parameter Store and [[appsync|Security]] Groups.

Explanation: While Systems Manager Parameter Store supports storing secrets, it does not support automatic rotation without additional customization. [[appsync|Security]] Groups are used for network access control and do not provide secret management capabilities.

Question 2: A global e-commerce platform wants to store sensitive information such as API keys and passwords in a secure manner while minimizing costs. They need to replicate these secrets across multiple regions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes. What strategy would best meet these requirements?

Correct Answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] to store the secrets and replicate them manually or programmatically to each region.

Explanation: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] supports encrypted storage and automatic rotation of secrets. By default, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] stores secrets in a single region. However, secrets can be copied manually or programmatically to other regions using the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] API.

Incorrect Answer: Use Systems Manager Parameter Store with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]].

Explanation: While Systems Manager Parameter Store supports storing secrets, it does not support automatic rotation without additional customization. Additionally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] is designed for point-in-time recovery of [[ec2]] instances, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] databases rather than secrets replication.