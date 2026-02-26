**Advanced Architecture**

At its core, AWS Identity Center (formerly known as AWS Single Sign-On) is an integrated service that enables SSO to cloud applications using federation standards such as [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 and OpenID Connect (OIDC). It supports enterprise directory services like Microsoft Active Directory and AWS Managed Microsoft AD, as well as social identity providers like Google and Facebook.

Internally, Identity Center uses Service Accounts for managing permissions within the boundaries of your own AWS accounts without requiring individual [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles. This allows you to centralize access control across multiple AWS accounts while leveraging existing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

For global [[organizations]], Identity Center offers a single point of management for user identities and their access to various AWS Regions and accounts. However, it's essential to understand that each Region has its own set of AWS Identity Centers, which means cross-Region replication isn't supported natively. To overcome this limitation, consider setting up synchronization processes using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and [[step-functions|AWS Step Functions]].

**Comparison & Anti-Patterns**

| Service              | Use Case                                                          |
| -------------------- | ----------------------------------------------------------------- |
| AWS [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]             | Primarily for granting AWS API access; not designed for SSO        |
| AWS [[cognito]] User Pools| Best suited for mobile and web apps needing custom UI and UX flows|

Anti-pattern: Using Identity Center as a replacement for AWS [[cognito]] when the latter provides more features tailored to consumer-facing applications.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented in JSON format. For example, here's a policy allowing specific IP ranges:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "NotAction": "iam:*",
            "Resource": "*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceVpc": [
                        "192.168.0.0/16"
                    ]
                }
            }
        }
    ]
}
```

Cross-account access can be managed through organization Service Control [[policies]] (SCPs) at the top level. For instance, you could enforce [[mfa]] deletion of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities by creating an [[SCP]] like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyMfaDelete",
            "Effect": "Deny",
            "Actions": [
                "iam:*Mfa*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "BoolIfExistsNot": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Identity Center has throttling limits including 5 requests per second for most operations. If these limits are exceeded, responses will include error codes indicating throttling. Implement exponential backoff strategies in your application code to handle such cases.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve distributing the load across multiple regions and ensuring data replication. As mentioned earlier, since Identity Center does not support native cross-region replication, alternative solutions involving [[lambda]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] should be considered.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by applying tags to resources associated with Identity Center. [[billing|Cost Explorer]] allows you to analyze costs based on these tags. Remember, there's no direct charge for using Identity Center; however, associated services like AWS Managed Microsoft AD may incur charges.

**Professional Exam Scenarios**

Scenario 1: A large financial institution wants to implement Identity Center but needs to ensure all users must authenticate via [[mfa]] before performing any actions. How would you address this requirement?

Correct Answer: Enforce [[mfa]] usage through an Organization [[SCP]] restricting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] actions unless [[mfa]] is present.

Incorrect Answer: Rely solely on Identity Center's built-in [[mfa]] capabilities because they do not provide sufficient enforcement mechanisms.

Scenario 2: An e-commerce company wants to use Identity Center for SSO but also needs to collect additional information during sign-up beyond what's provided by standard IdPs. What service should they use instead?

Correct Answer: AWS [[cognito]] User Pools offer extensible UI components for capturing custom data during registration.

Incorrect Answer: Using Identity Center and configuring complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to force user input isn't scalable nor secure.