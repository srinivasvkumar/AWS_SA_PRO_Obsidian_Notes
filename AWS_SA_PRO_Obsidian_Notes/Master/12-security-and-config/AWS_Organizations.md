**Advanced Architecture**

[[organizations|AWS Organizations]] is a master payee account that enables management of multiple AWS accounts from a single location. It provides a unified dashboard for resource and policy management, enabling features like [[organizations|consolidated billing]], account creation, and Service Control [[policies]] (SCPs) to ensure [[appsync|security]] and compliance across all accounts.

Internally, [[organizations|AWS Organizations]] uses a hierarchical structure called "organizational units" (OUs) to group accounts based on specific needs. Each OU can have zero or more child OUs, allowing for complex organizational structures.

At the top level is the root, which contains all accounts within an organization directly or through nested OUs. Accounts can be associated with only one OU at a time, but an account in one OU can be moved to another OU without interruption.

Global scale is achieved by replicating the core functionality of [[organizations|AWS Organizations]] across all regions. This ensures low-latency interactions between the master payee account and its member accounts, regardless of their geographical distribution.

**Comparison & Anti-Patterns**

| Feature | [[organizations|AWS Organizations]] | Alternatives |
| --- | --- | --- |
| Multi-account strategy | Centralized management of multiple accounts | [[control-tower|AWS Control Tower]], AWS Single Sign-On |
| Quotas | Applies SCPs to limit services and actions | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, permissions boundaries, SCPs |
| Integration | Supports AWS Services Control Insights, AWS Personal Health Dashboard, etc. | [[config|AWS Config]], AWS [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], AWS [[cloudwatch]] Events |

Anti-patterns include using [[organizations]] as a replacement for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or permissions boundaries, as it does not provide fine-grained control over individual user actions. Also, it should not be used to manage resources within an account, as other services like AWS Resource Access Manager and [[service-catalog|AWS Service Catalog]] are better suited for this purpose.

**[[appsync|Security]] & Governance**

[[organizations|AWS Organizations]] supports advanced [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON snippets. For example, you can create a custom [[SCP]] to restrict access to specific AWS services or actions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "ec2:DescribeInstances",
                "elasticloadbalancing:DescribeLoadBalancers"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-account access is enabled using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles that allow trusted entities (users, roles, or accounts) to perform specific actions. For instance, to grant an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user in one account access to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in another account, you would create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the destination account with a trust relationship policy that allows the source account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::source-account-ID:user/source-user"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::destination-bucket/*"
        }
    ]
}
```
Organization SCPs enforce centralized [[control-tower|guardrails]] for your entire organization, ensuring consistent [[appsync|security]] and compliance postures.

**Performance & Reliability**

[[organizations|AWS Organizations]] has throttling limits to prevent abuse and maintain system stability. These limits include a maximum number of events per minute (EPM) and a maximum number of API calls per second (CPS). If these limits are exceeded, you may experience [[api-gateway|errors]] or increased latencies. To mitigate these issues, implement exponential backoff strategies to retry failed requests gradually, reducing the load on the service.

HA/DR patterns involve creating separate [[organizations]] in different regions to minimize the impact of regional outages. However, this approach requires additional management overhead and coordination between the [[organizations]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls in [[organizations|AWS Organizations]] include tag-based cost allocation, which assigns costs to specific tags on resources. Additionally, you can set up [[Budgets]] and alerts for each account in the organization, ensuring timely notifications when spending thresholds are reached.

Calculation examples:
- Tag-based cost allocation: Assume 10 [[ec2]] instances, each with a `Department` tag value of either `Marketing` or `Engineering`. Using AWS [[billing|Cost Explorer]], you can view costs broken down by tag values, providing insight into departmental spending.
- [[Budgets]] and alerts: Set a monthly budget of $5,000 for an account, triggering notifications when 50%, 80%, and 100% of the budget is consumed.

**Professional Exam Scenarios**

Scenario 1: A company wants to ensure that no account in their organization can launch [[ec2]] instances with more than 4 vCPUs. Which of the following options achieves this goal?

A) Create an [[SCP]] that denies the `ec2:RunInstances` action with a condition allowing instances with less than or equal to 4 vCPUs.
B) Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in each account that grants the `ec2:RunInstances` action with a condition allowing instances with less than or equal to 4 vCPUs.
C) Modify the [[organizations|AWS Organizations]] settings to restrict the number of vCPUs per account.
D) Implement [[trusted-advisor|AWS Trusted Advisor]] checks to notify when [[ec2]] instances exceed 4 vCPUs.

Correct Answer: A
Justification: SCPs are the only option to apply restrictions across all accounts in an organization. By denying the `ec2:RunInstances` action with a condition allowing instances with less than or equal to 4 vCPUs, the company can achieve the desired outcome.

Incorrect Answers:
B) [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles do not provide account-wide restrictions.
C) There is no such setting in [[organizations|AWS Organizations]].
D) Trusted Advisor checks provide recommendations, not restrictions.

---

Scenario 2: A company has two AWS accounts, one for development and one for production. They want to share data between these accounts securely. Which of the following options meets their requirements?

A) Create an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in the development account, enable cross-account access, and grant permission to the production account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.
B) Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the development account, grant necessary permissions, and allow the production account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] principal to assume the role.
C) Create an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in the production account, enable cross-account access, and grant permission to the development account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.
D) Create a [[lambda]] function in the development account, grant necessary permissions, and allow the production account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] principal to invoke the function.

Correct Answer: B
Justification: By creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the development account with the necessary permissions and allowing the production account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] principal to assume the role, the company can securely share data between the two accounts.

Incorrect Answers:
A) This approach shares data only within [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], limiting flexibility.
C) The development environment should not have direct access to production data.
D) [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] are not designed for sharing data between accounts.