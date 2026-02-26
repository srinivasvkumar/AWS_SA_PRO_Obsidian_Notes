**Advanced Architecture**

At its core, [[control-tower|AWS Control Tower]] is a service that helps [[organizations]] create and manage multi-account environments using [[iam|best practices]]. It provides several features such as [[control-tower|account factory]], blueprints, and [[control-tower|guardrails]]. The underlying architecture consists of the following components:

* **Service Control Policy ([[SCP]]) service**: This is an [[organizations]] service that enables centrally managed policy actions across multiple accounts in an organization.
* **AWS Single Sign-On (SSO)**: A service that allows users to sign in to multiple AWS accounts with one set of credentials.
* **AWS [[appsync|Security]] Token Service ([[sts]])**: A service used by Control Tower to generate temporary [[appsync|security]] credentials for users and roles.
* **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]]**: A service used by Control Tower to automate the creation of resources such as [[organizations|AWS Organizations]], SSO, and Service Control [[policies]].

Control Tower uses these components to implement features like the [[control-tower|Account Factory]], which creates new member accounts in an organization, and blueprints, which define a specific configuration for a set of accounts. [[control-tower|Guardrails]] are implemented as Service Control [[policies]] and prevent the creation or modification of certain resources in the accounts.

Global scale is achieved through the use of AWS Regions and Availability Zones. Each region has at least three availability zones, and each Control Tower instance can be created in a single region. This means that Control Tower instances in different regions cannot communicate directly with each other, but they can still enforce Service Control [[policies]] across all accounts in the organization.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[control-tower|AWS Control Tower]] | Creating and managing multi-account environments using [[iam|best practices]] |
| [[organizations|AWS Organizations]] | Managing [[billing]], permissions, and [[appsync|security]] across multiple accounts |
| [[service-catalog|AWS Service Catalog]] | Creating and managing catalogs of approved products and services |

Anti-patterns include using Control Tower for a single account environment, or using it without enabling Service Control [[policies]]. Another anti-pattern is creating custom scripts to perform tasks already provided by Control Tower, such as creating new accounts.

**[[appsync|Security]] & Governance**

Control Tower implements [[appsync|security]] and governance through Service Control [[policies]], which are applied to all accounts in the organization. Here's an example of a JSON policy that prevents the deletion of Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "s3:DeleteBucket"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. For example, a role in one account can be assumed from another account, allowing users in the second account to perform actions in the first account.

Organization SCPs can also be used to restrict access to specific AWS services, or to limit the actions that can be performed on those services. For example, the following [[SCP]] denies the `ec2:TerminateInstances` action on all [[ec2]] instances:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "ec2:TerminateInstances"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**Performance & Reliability**

Throttling limits in Control Tower depend on the number of accounts in the organization. The limits for Service Control [[policies]] are 500 [[policies]] per organization, and 10 [[policies]] per account. If these limits are exceeded, requests will be throttled. To avoid throttling, use exponential backoff strategies when making API calls.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns can be implemented by creating Control Tower instances in multiple regions. This allows for failover in case of a regional outage. However, since Control Tower instances in different regions cannot communicate directly with each other, Service Control [[policies]] must be created in each region separately.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented by using AWS [[billing|Cost Explorer]] and [[billing|AWS Budgets]]. [[billing|Cost Explorer]] allows you to view your costs by account, tag, or service, while [[billing|AWS Budgets]] allow you to set [[Budgets]] and alarms for specific cost thresholds.

Here's an example calculation for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]:

Assume a multi-account environment with 10 accounts, and a monthly cost of $10,000. Using AWS [[billing|Cost Explorer]], we find that 80% of the cost comes from [[ec2]] instances. We then use [[billing|AWS Budgets]] to set a budget of $8,000 for [[ec2]] instances, and receive notifications if the cost exceeds this amount. By optimizing our [[ec2]] instances, we can reduce our monthly cost by 20%, saving $2,000 per month.

**Professional Exam Scenarios**

Scenario 1: A company wants to use Control Tower to manage their multi-account environment, but they have existing custom scripts for creating new accounts. Should they continue using these scripts or switch to Control Tower?

Correct answer: They should switch to Control Tower, as it provides built-in functionality for creating new accounts, reducing the need for custom scripts.

Incorrect answer: They should continue using their custom scripts, as they may not be compatible with Control Tower.

Scenario 2: A company wants to implement Service Control [[policies]] to restrict access to specific AWS services. Which of the following options is correct?

a) Create Service Control [[policies]] in each individual account.
b) Create Service Control [[policies]] in the master account and apply them to all accounts.

Correct answer: b) Create Service Control [[policies]] in the master account and apply them to all accounts.

Incorrect answer: a) Create Service Control [[policies]] in each individual account. This would require manually applying the same [[policies]] to every account, increasing the risk of [[api-gateway|errors]] and inconsistencies.