**Advanced Architecture**

At its core, [[cloudformation]] is a service that allows you to create and manage AWS resources using templates written in JSON or YAML. The templates define the desired state of your infrastructure as code, enabling version control and automation. However, advanced users require more than just creating individual [[cloudformation|stacks]]. They need to address multi-account strategies, quotas, and complex [[api-gateway|integrations]].

To achieve this, leverage [[cloudformation|nested stacks]], stack sets, and resource importing features. [[cloudformation|Nested stacks]] allow creating child [[cloudformation|stacks]] from a single parent stack, promoting reusability and modularity. [[cloudformation|StackSets]] extend this concept by deploying the same stack across multiple accounts and regions. Lastly, resource importing enables adding existing resources into a new or existing stack, facilitating gradual migration to [[cloudformation]].

[[RDS_Instance_Types|Global scale considerations]] involve understanding how changes propagate within and outside a stack. For instance, updating a resource with a global scope (like an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket) may have unintended consequences if not appropriately managed. To mitigate these issues, use the `DeletionPolicy` attribute to specify what should happen when the stack is deleted or updated. Also, be aware of drift detection, which identifies differences between the actual state of the environment and the template-defined configuration.

**Comparison & Anti-Patterns**

| Service          | When to use                                                              |
|-----------------|-------------------------------------------------------------------------|
| [[cloudformation]]  | Standardized, repeatable infrastructure; supports all AWS services      |
| Terraform       | Multi-cloud/infrastructure support; versioned granular dependencies   |
| CDK             | Higher level abstraction; easier learning curve; type safety           |

Common anti-patterns include mixing different resource types within a single file without proper namespacing, leading to confusion and potential conflicts. Another pitfall is overuse of [[cloudformation|custom resources]], which can lead to brittle solutions and tight coupling between components.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] often involve attaching permissions to specific roles or users. Here's an example granting admin access to a user within a [[cloudformation]] template:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

Cross-account access requires familiarity with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and their associated trust [[policies]]. Additionally, [[organizations]] can enforce [[appsync|security]] controls through Service Control [[policies]] (SCPs) at the organization level. An example [[SCP]] restricting certain actions within [[cloudformation]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "cloudformation:CreateStack",
                "cloudformation:UpdateStack"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "cloudformation:Initiator": "${aws:userid}"
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Throttling limits vary depending on the operation performed. In general, it's essential to implement exponential backoff strategies when dealing with APIs prone to throttling. A simple example:

```yaml
Resources:
  MyResource:
    Type: 'Custom::MyResource'
    Properties:
      ServiceToken: !GetAtt MyResourceService.Arn
      Operation: Create
    Metadata:
      aws_waiter_delay: 20
      aws_max_attempts: 6
      aws_waiter_max_delay: 20
```

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns depend on the use case but typically involve multi-region designs and [[cloudformation|change sets]] for controlled updates.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls come in various forms, such as limiting the number of resources created per account, setting tags for [[billing]] purposes, or leveraging cost estimation tools like the [[cloudformation]] Cost Estimation Welcome Console. Calculating costs requires understanding each resource's pricing model and considering factors like data transfer fees.

**Professional Exam Scenario**

Scenario 1: A company wants to manage their infrastructure across multiple accounts while maintaining visibility and consistency. What solution would you propose?

Correct answer: Leverage [[cloudformation]] [[cloudformation|StackSets]] along with Service Control [[policies]] (SCPs) at the organization level to ensure consistent configurations and enforce [[appsync|security]] [[policies]].

Incorrect answer: Using separate [[cloudformation]] templates for each account without proper sharing mechanisms could result in inconsistencies and difficulty managing resources centrally.

Scenario 2: A team needs to integrate their existing [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] with a [[cloudformation]] stack containing [[ec2]] instances. How would you approach this task?

Correct answer: Import the existing [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] into the [[cloudformation]] stack as a resource using the `AWS::EC2::VPC` type.

Incorrect answer: Creating a new [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] within the [[cloudformation]] template would cause disruption to the current network setup and potentially impact other dependent resources.