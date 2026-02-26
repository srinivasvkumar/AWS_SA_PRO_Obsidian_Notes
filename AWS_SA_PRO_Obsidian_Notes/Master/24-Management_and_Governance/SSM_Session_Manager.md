### Advanced Architecture

At its core, [[ssm|AWS Systems Manager (SSM)]] Session Manager is a fully managed service that helps you to automate tasks across your fleet of Amazon Elastic Compute Cloud ([[ec2]]) instances and hybrid servers using a secure, interactive command-line interface. It eliminates the need to open inbound ports or manage SSH keys, thus simplifying instance management.

Internally, [[ssm]] Session Manager uses AWS SDKs and the [[ssm]] agent, which runs on the [[ec2]] instances or hybrid servers. The [[ssm]] Agent communicates with the [[ssm]] service through HTTPS without having any exposed inbound ports. This architecture allows for a more secure and scalable approach as compared to traditional remote server management tools.

Global scale is achieved by utilizing regional endpoints and leveraging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. For example, when using Session Manager with hybrid servers, an [[ALB]] can be placed in multiple regions, and traffic from [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] will be routed to the nearest [[ALB]] based on the user location.

### Comparison & Anti-Patterns

| Service               | When to use                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| **Session Manager**   | Manage [[ec2]] instances and hybrid servers securely via CLI/browser           |
| **Systems Manager Run Command** | Remotely execute commands on multiple instances at once                        |
| **AWS [[ssm]] Parameter Store** | Securely store and retrieve application configuration data                    |
| **AWS [[ssm|SSM Patch Manager]]** | Automate OS patching across large numbers of instances                      |
| **AWS [[ssm]] Fleet Manager** | Perform maintenance activities like patching, updating, and installing software on instances |

Anti-pattern: Using Session Manager directly for managing AMIs, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances, or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. These resources should be managed using their respective APIs, SDKs, or AWS Management Console.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can include resource-level permissions, context-aware [[cloudformation|conditions]], and principal-based controls. Here's an example JSON policy allowing a specific user to start a session only on instances tagged "Environment":
```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:DescribeSessions",
    "ssm:StartSession"
  ],
  "Resource": [
    "*"
  ],
  "Condition": {
    "StringEqualsIfExists": {
      "aws:ResourceTag/Environment": [
        "Production"
      ]
    }
  },
  "Principal": {
    "AWS": [
      "arn:aws:iam::123456789012:user/johndoe"
    ]
  }
}
```
Cross-account access can be granted by creating a role in the source account, attaching necessary permissions, and then assuming it from the destination account. Organizational Service Control [[policies]] (SCPs) can enforce restrictions on usage, ensuring compliance with organizational standards.

### Performance & Reliability

Throttling limits vary depending on the operation performed. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], exponential backoff strategies can be implemented using the `aws-sdk` library, which automatically handles retries and backoffs.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], ensure that your instances are spread across different Availability Zones within a region. In case of a regional outage, use hybrid servers or create a secondary [[dr]] environment in another region.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be applied by using AWS [[billing|Cost Explorer]] to analyze costs associated with [[ssm]] Session Manager. Calculate the number of sessions per resource type and optimize accordingly. By default, there are no additional costs for using Session Manager, but charges may apply if using hybrid servers or executing operations that consume [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]].

### Professional Exam Scenarios

#### Scenario 1: Multi-Account Strategy

Imagine an organization with two separate AWS accounts: Account A and Account B. Both teams need to manage instances in Account A using [[ssm]] Session Manager. Create a cross-account access solution using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

Correct answer:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A allowing required permissions for [[ssm]] Session Manager.
2. Attach the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to the instances in Account A.
3. In Account B, create a trust relationship between the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users and the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A.
4. Users in Account B can now assume the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A to perform [[ssm]] Session Manager operations.

Incorrect answers:

1. Sharing the [[ssm]] Agent itself between accounts: Agents cannot be shared between AWS accounts because they are tied to the underlying instance.
2. Configuring a single AWS SSO setup for both accounts: While possible, this method does not provide granular control over [[ssm]] Session Manager permissions.

#### Scenario 2: Large-Scale Instance Patching

An enterprise has thousands of instances distributed across several regions. They want to use [[ssm|SSM Patch Manager]] to apply patches consistently while minimizing operational overhead.

Correct answer:

1. Implement infrastructure-as-code tools like TerraForm, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]], or [[AWS CDK]] to automate the creation and management of instances.
2. Enable automatic patching for newly created instances using [[ssm|SSM Patch Manager]].
3. Configure [[ssm|SSM Patch Manager]] maintenance windows to schedule regular patching.
4. Monitor patching progress using AWS [[ssm]] Reports.

Incorrect answers:

1. Using [[ssm|SSM Run Command]] to manually run patching scripts on each instance: This approach lacks scalability and requires manual intervention for every patch cycle.
2. Applying patches directly to Amazon Machine Images (AMIs): Modifying an AMI impacts all instances launched from that image, potentially causing disruption or compatibility issues.