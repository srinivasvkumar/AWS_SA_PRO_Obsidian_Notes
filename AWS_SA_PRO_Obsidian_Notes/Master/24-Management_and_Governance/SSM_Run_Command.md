### Advanced Architecture

At its core, [[AWS Systems Manager]] Run Command is a service that allows you to automate common administrative tasks by running scripts on a set of managed instances. The [[RDS_Instance_Types|internals]] involve an agent called the [[ssm]] Agent that is installed on each instance, which communicates with the Systems Manager service using the HTTPS protocol. This communication enables command execution, state management, patch management, and [[transfer-family|other features]].

Systems Manager Run Command supports global scaling through regional endpoints backed by Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[dynamodb]], and [[cloudwatch]] Events. The service uses these services in the background to manage commands, store metadata, and trigger events. For multi-region deployments, you can use [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or custom solutions to replicate data across regions.

### Comparison & Anti-Patterns

| Service               | Use Case                                                             |
| --------------------- | ------------------------------------------------------------------- |
| **Run Command**       | Manage Windows / Linux instances at scale                           |
| **[[step-functions|AWS Step Functions]]** | Coordinate multiple AWS services into serverless workflows         |
| **[[aws-batch|AWS Batch]]**          | Run batch computing workloads on AWS                              |
| **[[lambda|AWS Lambda]]**        | Short-lived, event-driven compute resources                        |

Anti-pattern example: Running long-running processes directly from Run Command. Instead, consider using [[aws-batch|AWS Batch]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]] for more complex tasks.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Run Command include restricting access to specific resources, resource types, and actions. Here's an example JSON policy granting access to Run Command:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:DescribeAssociation",
                "ssm:GetDeployablePatchSnapshotForInstance",
                "ssm:GetDocument",
                "ssm:DescribeDocument",
                "ssm:GetManifest",
                "ssm:GetParameter",
                "ssm:GetParameters",
                "ssm:ListAssociations",
                "ssm:PutInventoryItem",
                "ssm:PutComplianceItems",
                "ssm:PutConfigurePackageResult",
                "ssm:UpdateAssociationStatus",
                "ssm:UpdateInstanceAssociationStatus",
                "ssm:UpdateInstanceInformation"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access is possible via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. Organizational Service Control [[policies]] (SCPs) can enforce restrictions on [[ssm]] usage across accounts.

### Performance & Reliability

Run Command enforces throttling limits based on the number of concurrent sessions per region and account. To handle high loads, implement exponential backoff strategies when handling API [[api-gateway|errors]].

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include distributing instances across different Availability Zones and regions. Ensure proper configuration of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and failover configurations.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include setting up SCPs to limit [[ssm]] usage within your organization. Additionally, monitor [[ssm]] usage through [[cloudwatch]] metrics and alarms. Calculate costs using the following formula:

Cost = Number of commands \* (Command cost + Instance cost)

### Professional Exam Scenario

**Scenario 1:** Your company manages a large fleet of [[ec2]] instances distributed across several regions. You need to execute a PowerShell script remotely on all Windows instances to update a piece of software. Which solution provides the most efficient and secure approach?

Correct answer: Implement an [[AWS Systems Manager]] Run Command document with the required PowerShell script. Then, create an association with the target instances and execute the command.

Incorrect answer: Directly RDP into each instance and run the PowerShell script manually. This method does not allow for easy scalability and monitoring.

**Scenario 2:** A development team needs to invoke an [[AWS Systems Manager]] Run Command from their application. However, they do not want to embed credentials in their code. How should they achieve this without compromising [[appsync|security]]?

Correct answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with permissions to execute Systems Manager Run Command. Attach the role to the [[ec2]] instances where the application runs.

Incorrect answer: Store credentials in environment variables or a configuration file. These methods may expose sensitive information if accessed by unauthorized users.