**Advanced Architecture: [[organizations]] [[organizations|Consolidated Billing]]**

[[organizations]] is a service that enables creation of hierarchical organization structures, management of multiple AWS accounts, and applying [[policies]] across them. It also provides [[organizations|consolidated billing]] feature which aggregates costs from all member accounts into a single payment method.

Internally, [[organizations]] uses a centralized account called "master payer" that handles the [[billing]] and pays for all the usage in the member accounts. The master payer can have one or more payment methods attached like credit cards or AWS managed bills.

[[RDS_Instance_Types|Global scale considerations]] include [[organizations]] being available in all public regions except China and GovCloud. Multi-region support is achieved by replicating certain configurations such as Service Control [[policies]] (SCPs) but not the entire hierarchy.

Under the hood, [[organizations]] maintains a separate set of metadata about each member account including its status, type (master/member), and the root it belongs to. This information is stored in [[dynamodb|DynamoDB tables]] while SCPs are managed using AWS [[sts]] AssumeRole APIs.

**Comparison & Anti-Patterns**

| Issue | Solution |
|---|---|
| Centralized control over many AWS accounts | [[organizations]] with [[organizations|Consolidated Billing]] |
| Separate [[billing]] for different departments | Using multiple [[organizations]] |
| Full automation of resource provisioning | Use [[cloudformation]] [[cloudformation|StackSets]] instead |
| Account sharing without proper [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles | Use AWS Resource Access Manager |

Common anti-pattern includes using [[organizations]] solely for cost allocation purposes without leveraging its policy enforcement capabilities.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing specific actions at various levels within the hierarchy. For example, granting `organizations:CreateOrganization` action only at the root level:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "organizations:CreateOrganization",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "organizations:MasterAccountId": "<master-account-id>"
                },
                "Bool": {
                    "organizations:BypassPolicyLockoutSafeMode": "true"
                }
            }
        }
    ]
}
```
Cross-account access involves creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the trusted account and specifying the principal as the Organization's master account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<master-account-id>"
            },
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```
Service Control [[policies]] (SCPs) can be used to enforce [[control-tower|guardrails]] across all member accounts. For instance, restricting [[ec2]] instances to run within a particular region:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "NotAction": "ec2:RunInstances",
            "Resource": "*",
            "Condition": {
                "StringNotEqualsIgnoreCase": {
                    "ec2:Region": ["us-west-2"]
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Throttling limits exist primarily on the number of requests per second. If exceeded, [[organizations]] will return a `429 Too Many Requests` error. To mitigate this, implement exponential backoff strategies with jitter.

High availability/disaster recovery patterns typically involve having secondary master payer accounts in different regions. However, since [[organizations]] does not support multi-region hierarchies, this pattern cannot be implemented directly. Instead, manual synchronization could be used.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved through tagging resources and enabling [[billing|cost explorer]] at the master payer level. [[billing|Cost Explorer]] then aggregates costs based on these tags.

For example, given two tagged [[ec2]] instances:

- Instance A: Type=t2.micro, Region=us-east-1, Tag=Team:Dev
- Instance B: Type=t2.small, Region=us-west-2, Tag=Team:QA

Their costs would appear separately in [[billing|Cost Explorer]] under the respective teams.

**Professional Exam Scenarios**

Scenario 1: An enterprise wants to enforce [[appsync|security]] [[policies]] across multiple AWS accounts. They currently use [[config|AWS Config]] rules but find them difficult to manage centrally. How would you address this issue?

Correct Answer: Migrate to [[organizations]] and apply Service Control [[policies]] (SCPs) to enforce the desired [[appsync|security]] [[policies]] across all member accounts.

Incorrect Answer: Continue using [[config|AWS Config]] rules and create a [[lambda]] function to aggregate findings from each account.

Scenario 2: A startup has been using AWS for several months and now needs better visibility into their spending habits. They want to allocate costs to different departments and track them individually. What steps should they follow?

Correct Answer: Create separate AWS accounts for each department, associate them with an [[organizations|AWS Organizations]] master payer account, and enable [[organizations|Consolidated Billing]].

Incorrect Answer: Keep using a single AWS account and add tags to resources for cost allocation.