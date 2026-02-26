**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[ec2]] Image Builder is a fully managed service that makes it easy to automate the creation, maintenance, and deployment of custom Windows and Linux Amazon Machine Images (AMIs). It supports multi-account strategies through the use of components, recipes, and image build variant versions. An Image Builder "image pipeline" defines the workflow for creating images, including the source AMI, the components to be added or updated, the testing steps, and the output AMIs. Image pipelines can be created in one account and shared across multiple accounts using Service Control Policy ([[SCP]]) permissions.

Internally, [[ec2]] Image Builder uses [[AWS Systems Manager]] Change Calculator to determine the differences between two sources, such as an existing AMI and a new version of a component. This allows for efficient updates of only the changed files rather than a full copy of the source AMI. The service also utilizes Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storing temporary artifacts, and Amazon [[cloudwatch]] for [[vpc-flow-logs|logging]] and monitoring.

Global scale is achieved by running Image Builder within each supported region, allowing for localized image builds. However, there is no built-in support for cross-region replication or synchronization of images.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Use Case | [[ec2]] Image Builder | Packer | Golden AMI |
|---|---|---|---|
| Automated image building | Yes | Yes | Yes |
| Multi-account support | Through SCPs | Limited | Manual sharing |
| Component-based modularity | Yes | Limited | Monolithic |
| Versioning and history tracking | Yes | Limited | Manual |

Anti-patterns include:

* Using Image Builder for non-image tasks, like configuration management or patching.
* Not following the principle of least privilege when granting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] permissions.
* Failing to monitor and alert on image build failures or other [[api-gateway|errors]].

**[[RDS_Instance_Types|3. Security & Governance]]**

Image Builder supports complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON, allowing granular control over resources and actions. For example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeImages",
                "ec2:CreateImage",
                "ec2:DeregisterImage"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access can be granted via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, allowing principals from one account to perform actions in another account. Additionally, Service Control [[policies]] (SCPs) at the organization level can enforce centralized control over [[ec2]] Image Builder usage.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[ec2]] Image Builder has throttling limits based on the number of concurrent image builds and API calls. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies when encountering throttled responses. High availability/disaster recovery (HA/DR) patterns should involve building images in multiple regions, either manually or using infrastructure as code tools.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls in Image Builder are limited to selecting specific instance types for image builds. To calculate costs, use the AWS Pricing Calculator or the AWS [[billing|Cost Explorer]] tool. Keep in mind that storage costs associated with snapshots and AMIs will add to the overall cost.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You are tasked with implementing a solution to automate image building across multiple AWS accounts. Which services would you leverage?

Correct answer: [[ec2]] Image Builder and [[organizations|AWS Organizations]], utilizing Service Control [[policies]] (SCPs) for cross-account permission management.

Incorrect answer: [[cicd|AWS CodeBuild]] and [[cicd|AWS CodePipeline]], which do not provide native support for image building.

Scenario 2:
Your organization requires a highly available image building solution, with images built in both US East (N. Virginia) and Europe (Ireland) regions. How would you ensure high availability while minimizing operational overhead?

Correct answer: Implement separate [[ec2]] Image Builder pipelines in each region, triggered manually or via infrastructure as code tools.

Incorrect answer: Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] to replicate images between regions, adding unnecessary complexity and potential data transfer costs.