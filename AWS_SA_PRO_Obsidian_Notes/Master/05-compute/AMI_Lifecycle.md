### Advanced Architecture
At its core, Amazon Machine Images (AMIs) are virtual appliances that provide the necessary information to launch an instance in AWS. This includes details about the underlying operating system, any installed applications, and launch permissions. AMIs can be shared across accounts and regions, enabling sophisticated architectures that span multiple AWS accounts and even continents.

When designing AMI lifecycles at scale, it'
s essential to understand the [[RDS_Instance_Types|internals]] of AMIs. Each AMI consists of four fundamental components:

- **Name and Description**: Human-readable strings that help identify the AMI.
- **Image Location**: The location within Amazon Simple Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]) where the image resides.
- **Launch Permissions**: Control which AWS accounts or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users can launch instances from the AMI.
- **Block Device Mapping**: Describes the volumes available when launching an instance.

AMIs support various encrypted configurations, including encrypted root devices and snapshots. Additionally, AMIs created from Amazon Elastic Block Store ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]])-backed instances enable you to specify the [[kms]] key used for encryption.

Global scaling is achieved through AWS services like AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] and [[organizations|AWS Organizations]]. [[ram]] enables customers to share their resources, such as AMIs, across different AWS accounts and regions. In contrast, [[organizations|AWS Organizations]] simplify the management of multiple AWS accounts by providing centralized management of [[policies]], permissions, and [[billing]].

### Comparison & Anti-Patterns
The following table compares the primary use cases of AMIs against alternative solutions like Docker containers and [[ec2]] Systems Manager:

| Use Case                      | AMI                | Docker Container   | [[ec2]] System Manager |
|-------------------------------|--------------------|-------------------|-------------------|
| Golden Image Management       | **Strongly Recommended**         | Not Recommended    | Partially Supported|
| Application Deployment        | Possible           | **Recommended**   | **Recommended**   |
| Configuration Drift Prevention | Partially Supported | Not Recommended    | **Recommended**   |

Common anti-patterns include:

- Using AMIs for configuration drift prevention rather than using tools like [[ec2]] System Manager or Puppet.
- Relying solely on AMIs for application deployment without considering containerization technologies like Docker or AWS [[Fargate]].

### [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for AMIs often involve JSON structures similar to the following example, which allows cross-account sharing:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [
        "ec2:DescribeImages",
        "ec2:CreateImage",
        "ec2:ModifyImageAttribute",
        "ec2:CopyImage",
        "ec2:RegisterImage",
        "ec2:DeregisterImage"
      ],
      "Resource": "arn:aws:ec2:*:*:image/*"
    }
  ]
}
```
Cross-account access requires careful consideration of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, permissions, and Organization Service Control [[policies]] (SCPs). For instance, [[organizations]] may enforce tightened control over AMIs by implementing SCPs that prevent unauthorized modifications or deletions.

### Performance & Reliability
Throttling limits for AMIs vary depending on the API operation. For example, the maximum number of DescribeImages requests per second is 50. To manage throttling issues, implement exponential backoff strategies using libraries like the AWS SDK for Python (Boto3).

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns depend on the specific use case. However, [[iam|best practices]] include deploying instances in multiple Availability Zones (AZs) and replicating critical data across regions using services like [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]].

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls for AMIs involve understanding how each component contributes to overall costs. Specifically, consider the following factors:

- Number of unique AMIs
- Volume of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots
- Regions where AMIs are stored
- Size and type of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes associated with instances launched from AMIs

Calculation examples might include estimating the monthly storage costs for an AMI based on its size, snapshot frequency, and geographical distribution.

### Professional Exam Scenarios
#### Scenario 1
Suppose your organization has a strict requirement for storing all AMIs containing sensitive data in a single region while maintaining global accessibility. How would you address this challenge?

*Correct Answer*: Implement AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to centrally manage and distribute AMIs across accounts and regions. By doing so, you can store the actual AMI content in a single region while allowing other AWS accounts to access them seamlessly.

Incorrect Answers:

- Storing AMIs in multiple regions: This solution introduces redundancy and increases costs without addressing the requirement for centralized storage.
- Encrypting AMIs: While encryption is important for [[appsync|security]] purposes, it does not directly address the need for centralized storage.

#### Scenario 2
As part of your company's DevOps process, developers frequently create custom AMIs for testing purposes. Over time, these AMIs accumulate, leading to confusion and increased costs. Propose a strategy to mitigate this issue.

*Correct Answer*: Implement an automated AMI retirement policy using [[lambda|AWS Lambda]] and Amazon [[eventbridge]]. Set up a rule that triggers the [[lambda]] function daily to [[nonportable_instructions|review]] unused AMIs based on predefined criteria, such as age or lack of recent usage. If an AMI meets the specified [[cloudformation|conditions]], the [[lambda]] function sends a notification to the appropriate team member responsible for managing the AMI.

Incorrect Answers:

- Manually reviewing and deleting AMIs: This approach is error-prone, time-consuming, and unsustainable as the number of AMIs grows.
- Disabling AMI creation altogether: This solution prevents developers from creating new AMIs but does not address the issue of existing, unnecessary AMIs.