Advanced Architecture
---------------------

At its core, Copilot CLI is a command-line interface that enables developers to interact with various AWS services in a unified manner. It achieves this by using the concept of "flights," which are sequences of tasks that can be executed across multiple accounts and regions. Each flight can include one or more "operations" that perform specific actions on AWS resources.

Internally, Copilot CLI uses the AWS SDKs and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] to manage resources. This allows it to support a wide variety of AWS services and handle complex deployments across multiple accounts and regions. Additionally, Copilot CLI supports "blue/green" deployments, enabling developers to test new versions of their applications without disrupting the production environment.

[[RDS_Instance_Types|Global Scale Considerations]]
----------------------------

When deploying resources at a global scale, Copilot CLI provides several features to ensure high availability and low latency. For example, it supports Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] for DNS routing and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] for content delivery. Copilot CLI also supports multi-region deployments, allowing developers to deploy resources in multiple regions for improved performance and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].

Under the Hood Mechanics
-------------------------

Copilot CLI uses the concept of a "virtual private network ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) peering connection" to enable secure communication between different AWS accounts. This allows developers to manage resources in separate accounts as if they were part of the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. Copilot CLI also supports [[parameter-store|AWS Systems Manager Parameter Store]] for storing configuration data, making it easy to manage secrets and other configuration data across multiple environments.

Comparison & Anti-Patterns
---------------------------

While Copilot CLI is a powerful tool for managing AWS resources, there are some cases where it may not be the best choice. Here are some anti-patterns and comparison points:

| Service | When to use | When NOT to use |
| --- | --- | --- |
| Copilot CLI | Managing resources across multiple accounts and regions | Simple deployments or deployments within a single account |
| [[AWS CDK]] | Infrastructure as code (IaC) development | Ad hoc management of AWS resources |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] | Creating and updating stack-based AWS resources | Custom resource creation or deployment |

[[appsync|Security]] & Governance
----------------------

Copilot CLI supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], enabling developers to control who can perform specific operations on AWS resources. Here's an example JSON policy that grants permission to create a new application using Copilot CLI:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "copilot:CreateApplication"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access can be achieved using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. For example, here's a trust policy that allows Copilot CLI to assume an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in another account:
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
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalOrgID": "o-123456789012"
                }
            }
        }
    ]
}
```
Organization Service Control [[policies]] (SCPs) can also be used to enforce [[appsync|security]] [[policies]] across all accounts in an organization.

Performance & Reliability
--------------------------

Copilot CLI has throttling limits in place to prevent overuse and ensure reliability. These limits can be adjusted using the `copilot limit` command. If a request exceeds these limits, Copilot CLI will return a `ThrottlingException`. To handle this, developers should implement exponential backoff strategies.

Here's an example of an exponential backoff strategy in Python:
```python
import time

def call_api():
    # Call Copilot CLI API
    pass

def handle_throttling_exception(e):
    print(f"Request was throttled: {e}")
    time.sleep(2**retries)

for retries in range(5):
    try:
        call_api()
    except ThrottlingException as e:
        handle_throttling_exception(e)
```
HA/DR patterns can be implemented using blue/green deployments, which allow developers to test new versions of their applications without disrupting the production environment.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Copilot CLI supports granular cost controls through the use of tags. Tags can be applied to AWS resources, allowing developers to track costs and set up [[billing]] alerts based on usage.

For example, here's how to add a tag to an [[lambda|AWS Lambda]] function:
```bash
copilot env init --name my-env \
  --org my-org \
  --application my-app \
  --auto-deploy \
  --vpc \
  --tag team=backend \
  --tag costcenter=1234
```
This creates an environment called `my-env` with the tags `team=backend` and `costcenter=1234`, which can then be used to calculate costs and set up [[billing]] alerts.

Professional Exam Scenarios
---------------------------

Scenario 1: A company needs to deploy a new application across multiple accounts and regions. They want to ensure high availability and low latency. Which AWS services should they use?

Correct Answer: Copilot CLI, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]], Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], and [[parameter-store|AWS Systems Manager Parameter Store]].

Justification: Copilot CLI can be used to manage resources across multiple accounts and regions. Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] can be used for DNS routing, while Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] can be used for content delivery. [[parameter-store|AWS Systems Manager Parameter Store]] can be used to store configuration data.

Incorrect Answers:

* Using only Copilot CLI would not provide low latency, as it does not handle DNS routing or content delivery.
* Using only Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] would not provide the ability to manage resources across multiple accounts and regions.

Scenario 2: A developer wants to deploy a new application using Copilot CLI, but they need to ensure that only certain users have access to specific operations. Which AWS service should they use?

Correct Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Justification: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be used to grant permissions to specific users for specific operations. For example, a policy could be created that grants permission to create a new application, but denies permission to delete resources.

Incorrect Answers:

* Using [[organizations|AWS Organizations]] SCPs would not provide fine-grained control over individual user permissions.
* Using [[AWS CDK]] would not provide the ability to control individual user permissions.