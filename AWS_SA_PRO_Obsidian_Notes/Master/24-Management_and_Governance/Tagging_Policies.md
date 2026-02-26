**Advanced Architecture: Tagging [[policies]] in AWS**

Tagging [[policies]] in AWS allow for advanced management and governance of resources across accounts. They provide a method for enforcing tag usage, propagating tags, and validating tag integrity. This is achieved through the use of tag key values, tag restrictions, and tag enforcement types.

*[[RDS_Instance_Types|Internals]]:*

1. *Tag Key Values*: These are used to identify specific resources or sets of resources within an account. For example, a tag key value of "Environment" could be used to identify resources that belong to a specific environment such as "Production", "Staging", or "Development".
2. *Tag Restrictions*: These are used to restrict the tag keys and values that can be applied to resources within an account. For example, a tag restriction could be set up to only allow the tag key "Environment" with the value "Production" to be applied to [[ec2]] instances.
3. *Tag Enforcement Types*: These are used to enforce the use of tags on resources within an account. There are two types of tag enforcement: *preventative* and *[[AWS_SA_PRO_Obsidian_Notes/Master/Security/Detective|detective]]*. Preventative tag enforcement prevents resources from being created without the required tags, while [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Detective|detective]] tag enforcement identifies resources that do not have the required tags after they have been created.

*[[RDS_Instance_Types|Global Scale Considerations]]:*

Tagging [[policies]] are applied at the account level and can be used to manage resources across multiple regions. However, it is important to note that tagging [[policies]] are not automatically applied to new regions when they are added to an account. To apply tagging [[policies]] to new regions, they must be manually enabled in each region. Additionally, tagging [[policies]] are not supported for all resource types in all regions. It is important to check the AWS documentation for supported resource types and regions before implementing tagging [[policies]].

*Under the Hood Mechanics:*

Tagging [[policies]] are implemented using [[organizations|AWS Organizations]], Service Control [[policies]] (SCPs), and AWS Resource Groups.

1. *[[organizations|AWS Organizations]]*: This service allows for the creation of organizational units (OUs) and placement of accounts within these OUs. Tagging [[policies]] can be applied to OUs, allowing for centralized management of tagging [[policies]] across multiple accounts.
2. *Service Control [[policies]] (SCPs)*: These are used to define permissions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities (users, groups, roles) within an organization. SCPs can be used to enforce tagging [[policies]] by denying permissions to create resources without the required tags.
3. *AWS Resource Groups*: This service allows for the grouping of resources based on tags. Once resources are grouped, tagging [[policies]] can be applied to the entire group, making it easier to manage tags for large numbers of resources.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| Tagging [[policies]] | Centralized management of tags across multiple accounts and regions. Enforcing tag usage and propagating tags. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] | Fine-grained access control for [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entities. |
| Service Quotas | Setting and managing service limits for specific resource types. |

Anti-pattern: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to enforce tag usage instead of tagging [[policies]]. While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be used to enforce tag usage, they are not designed for this purpose and can lead to overly complex and difficult-to-manage [[policies]].

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for tagging [[policies]] involve the use of condition statements to enforce tag usage. For example, the following JSON policy snippet can be used to enforce the use of the tag key "Environment" with the value "Production" for [[ec2]] instances:
```json
{
    "Effect": "Deny",
    "Action": "ec2:RunInstances",
    "Resource": "*",
    "Condition": {
        "StringNotEqualsIfExists": {
            "ec2:LaunchTemplateData.Tags.Key": "Environment",
            "ec2:LaunchTemplateData.Tags.Value": "Production"
        }
    }
}
```
Cross-account access can be granted by attaching the same tagging policy to multiple accounts within an organization. Additionally, tagging [[policies]] can be applied to Organization Service Control [[policies]] (SCPs) to enforce tag usage across all accounts within the organization.

**Performance & Reliability**

Tagging [[policies]] have no direct impact on performance or reliability. However, improper implementation of tagging [[policies]] can lead to issues with resource creation and management. To avoid these issues, it is important to follow [[iam|best practices]] for tagging [[policies]] and regularly [[nonportable_instructions|review]] their implementation.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by using tagging [[policies]] to categorize resources and track costs. For example, tagging [[policies]] can be used to enforce the use of tag keys such as "Department" or "Project", allowing for easy identification of resource costs associated with specific departments or projects. Calculation examples:

1. Total monthly cost for the Marketing department: `(cost per EC2 instance) x (number of EC2 instances with tag "Department" = "Marketing")`
2. Total monthly cost for the Sales project: `(cost per RDS instance) x (number of RDS instances with tag "Project" = "Sales")`

**Professional Exam Scenarios**

Scenario 1: A company has multiple AWS accounts and wants to enforce the use of specific tag keys and values for certain resource types. The company also wants to ensure that these tags are propagated to child resources.

Correct answer: Implement tagging [[policies]] using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to enforce the use of specific tag keys and values. Use AWS Resource Groups to group resources based on tags and apply tagging [[policies]] to the entire group.

Incorrect answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to enforce tag usage and propagation. This approach is not designed for tag enforcement and can lead to overly complex and difficult-to-manage [[policies]].

Scenario 2: A company has multiple AWS accounts and wants to track costs for specific departments.

Correct answer: Implement tagging [[policies]] to enforce the use of a "Department" tag for all resources. Regularly [[nonportable_instructions|review]] tag usage and ensure that all resources have the required tag. Use AWS [[billing|Cost Explorer]] to view costs associated with specific tag values.

Incorrect answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to enforce tag usage and propagation. This approach does not allow for tracking costs associated with specific tag values.