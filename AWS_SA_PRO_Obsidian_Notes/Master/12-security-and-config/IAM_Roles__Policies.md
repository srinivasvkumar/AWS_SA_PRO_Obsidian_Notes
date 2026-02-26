**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles are AWS identities with specific permissions that allow access to AWS resources in your account. They are not associated with a specific user or group, but with permissions for a specific operation. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles can be assumed by authorized users, applications, or AWS services such as [[ec2]].

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role trust [[policies]] define who can assume the role. This is done through an Amazon Resource Name (ARN) that specifies which AWS principal can assume the role. The principal can be an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user, an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role, federated identity, or AWS service.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles can be used to delegate access to users in other accounts, allowing for cross-account access. This is particularly useful in multi-account environments, where one account might need to access resources in another account. To enable this, you create a trust policy that allows the source account to assume the role in the destination account.

Global scale is achieved through the use of [[organizations|AWS Organizations]]. With [[organizations]], you can apply Service Control [[policies]] (SCPs) at the organization level, which restrict or grant permissions across all accounts within the organization. This allows for centralized control over permissions and ensures consistent [[policies]] across all accounts.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Roles | Delegating access to users, applications, or AWS services |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Users | Granting individual users long-term access to AWS resources |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Groups | Managing permissions for multiple users |
| [[sts]] Assume Role | Allowing temporary access to resources in another account |

Anti-pattern: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users for cross-account access instead of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. This results in manual management of user permissions and increases operational complexity.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can lead to [[appsync|security]] risks if not properly managed. Here's an example of a JSON policy that grants read-only access to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::example-bucket/*",
                "arn:aws:s3:::example-bucket"
            ]
        }
    ]
}
```

Cross-account access is achieved using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with a trust policy that allows users from another account to assume the role. For example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```

Organization SCPs can enforce [[appsync|security]] [[policies]] across all accounts in an organization. For example, you could deny all access to a specific [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAccessToBucket",
      "Effect": "Deny",
      "Action": [
        "s3:*"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] has a throttling limit of 10 requests per second for authenticated API calls and 2 requests per second for unauthenticated API calls. If these limits are exceeded, the API calls will fail with a 429 Too Many Requests error. To handle this, implement exponential backoff strategies to retry failed requests.

HA/DR patterns for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] include using [[organizations|AWS Organizations]] to manage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] resources across multiple regions. By creating an organization and adding accounts to it, you can ensure that [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] are replicated across regions.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for cross-account access, which reduces the number of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users and groups needed. Additionally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles can be assigned to [[ec2]] instances, reducing the need for long-term access keys.

Calculation example: Assuming 100 users with 10 [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] each, the total cost would be $0. There are no charges for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: A company has two AWS accounts, one for production and one for staging. They want to grant read-only access to their [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets in the staging account from the production account. How would you accomplish this?

Correct answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the staging account with a trust policy that allows the production account to assume the role. Then, attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants read-only access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets. In the production account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user with permissions to assume the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.

Incorrect answer: Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users in the staging account with read-only access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets, and share the access keys with the production account. This approach is an anti-pattern because it requires manual management of user permissions and increases operational complexity.

Scenario 2: A company wants to enforce a [[appsync|security]] policy that denies access to a specific [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket across all accounts in their [[AWS Organization]]. How would you accomplish this?

Correct answer: Create an Organization [[SCP]] that denies all access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. Attach the [[SCP]] to the root of the organization, ensuring that it applies to all accounts.

Incorrect answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that denies access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and attach it to the relevant [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles. This approach does not provide centralized control over permissions and does not ensure consistent [[policies]] across all accounts.