**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[service-catalog|Service Catalog]] allows for the creation of portfolios containing products, which are services or applications that users can deploy into their own AWS accounts. Portfolios can be shared across accounts using [[organizations|AWS Organizations]].

Internally, Service CCatalog uses [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] under the hood for stack deployment. This means that each product in [[service-catalog|Service Catalog]] is essentially an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] stack. The templates used by these [[cloudformation|stacks]] can be versioned and managed in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

[[service-catalog|Service Catalog]] also supports provisioning artifacts, which allow for additional configuration options during deployment. These artifacts are written in either [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] macros or [[lambda|AWS Lambda]].

Global scale is achieved through the use of [[organizations|AWS Organizations]]. By creating a portfolio in one account and sharing it with other accounts in your organization, you can effectively provide a global catalog of services. However, [[service-catalog|Service Catalog]] itself does not have built-in global infrastructure.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
| --- | --- |
| [[service-catalog|Service Catalog]] | Centralized management and distribution of reusable services and applications. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] | Standalone infrastructure as code solution without centralized management capabilities. |
| AWS Elastic Beanstalk | Simplified application deployment and management, not well suited for centralized management. |

Anti-pattern: Using [[service-catalog|Service Catalog]] as a standalone service for managing infrastructure. [[service-catalog|Service Catalog]] should be used in conjunction with other services such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] and [[AWS Systems Manager]].

**[[RDS_Instance_Types|3. Security & Governance]]**

[[service-catalog|Service Catalog]] supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for controlling who can access which portfolios and products. For example, you can create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that allow users to view but not launch specific products.

Cross-account access is achieved through [[organizations|AWS Organizations]]. By sharing a portfolio with another account, users in that account can launch products from the shared portfolio. Additionally, [[service-catalog|Service Catalog]] supports cross-account access through AWS Single Sign-On.

Organization Service Control [[policies]] (SCPs) can be used to enforce restrictions on [[service-catalog|Service Catalog]] at the organization level. For example, you can use SCPs to prevent the creation of new portfolios in certain member accounts.

Example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "servicecatalog:DescribeProduct",
                "servicecatalog:DescribePortfolio"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": [
                "servicecatalog:CreateProduct",
                "servicecatalog:UpdateProduct"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[service-catalog|Service Catalog]] has throttling limits in place to prevent overuse. If you exceed these limits, you will receive a `429 Too Many Requests` error. To avoid this, you can implement exponential backoff strategies.

For high availability, [[service-catalog|Service Catalog]] recommends distributing your resources across multiple Availability Zones. For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], [[service-catalog|Service Catalog]] recommends backing up your [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] templates and other associated resources.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls in [[service-catalog|Service Catalog]] can be implemented by tagging products and portfolios. This allows you to track costs and set up [[billing]] alarms based on those tags.

Additionally, [[service-catalog|Service Catalog]] supports tagging of launched products, allowing you to track costs at the individual product level.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: A company wants to share a set of approved AMIs with its development teams. The development teams should be able to launch instances from these AMIs, but they should not be able to modify the original AMIs.

Correct answer: Create a [[service-catalog|Service Catalog]] portfolio containing the approved AMIs, and share it with the development teams. Set the launch constraint to "Immutable".

Incorrect answer: Share the AMIs directly with the development teams. This would allow the developers to modify the original AMIs.

Scenario 2: A company wants to restrict the use of [[service-catalog|Service Catalog]] to specific IP addresses.

Correct answer: Implement a Service Control Policy ([[SCP]]) in [[organizations|AWS Organizations]] to restrict access to [[service-catalog|Service Catalog]] based on IP address.

Incorrect answer: Implement network ACL rules to restrict access to [[service-catalog|Service Catalog]]. This would only restrict access to the [[service-catalog|Service Catalog]] API, not the [[service-catalog|Service Catalog]] web interface.