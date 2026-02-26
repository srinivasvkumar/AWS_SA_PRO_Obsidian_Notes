### Advanced Architecture
At its core, [[ssm|AWS Systems Manager (SSM)]] Patch Manager is a service that helps you automate the process of patching your Amazon [[ec2]] instances and hybrid servers. It leverages AWS [[ssm]] capabilities such as Session Manager, Maintenance Windows, and Patch Baselines to enable easy deployment and management of patches across various operating systems.

Internally, [[ssm|SSM Patch Manager]] operates by using AWS [[ssm]] Agent installed on each managed instance. The agent communicates with the [[ssm]] service, reporting the current patch compliance status and applying patches when necessary. Global scale is achieved through regional endpoint availability, allowing you to manage instances in multiple regions from a single account.

### Comparison & Anti-Patterns
The following table highlights some key differences between [[ssm|SSM Patch Manager]] and other services like [[cicd|AWS CodePipeline]], [[cicd|AWS CodeBuild]], and [[cicd|AWS CodeDeploy]]:

| Service               | Primary Function                   | Supports Patching?    |
|-----------------------|-------------------------------------|----------------------|
| [[ssm|SSM Patch Manager]]     | Instance patching & maintenance    | Yes                  |
| CodePipeline          | Continuous delivery orchestration  | No                   |
| CodeBuild             | Compile source code into artifacts   | No                   |
| CodeDeploy           | Application deployment              | No                   |

Anti-patterns include using [[ssm|SSM Patch Manager]] for deploying application artifacts or compiling source code. Instead, these tasks should be handled by services like CodePipeline, CodeBuild, and CodeDeploy.

### [[appsync|Security]] & Governance
To ensure secure cross-account access, you can create an [[ssm|SSM Patch Manager]] document that references an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in another account. This allows the patch manager to perform actions on resources in the second account. For organization-wide governance, you can enforce specific patch baseline rules using Service Control [[policies]] (SCPs) at the organization level.

Example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy for cross-account access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:DescribeInstanceInformation",
        "ssm:GetDeployablePatchSnapshotForInstance",
        "ssm:DescribeMaintenanceWindowTarget",
        "ssm:DescribeMaintenanceWindows"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpc": "vpc-xxxxxxx"
        }
      }
    }
  ]
}
```

### Performance & Reliability
When working with [[ssm|SSM Patch Manager]], it's crucial to understand throttling limits and implement appropriate backoff strategies. As per the [official documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-patch-throttle.html), there is a limit of 10 concurrent patching operations per region per account. To mitigate potential issues, implement exponential backoff strategies when encountering rate limits.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve distributing instances across different Availability Zones and regions. By doing so, you ensure that patching does not impact system uptime.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be implemented by setting up patching schedules based on maintenance windows. This way, you avoid unnecessary downtime while optimizing resource utilization. Additionally, you can set up [[billing]] alarms to monitor costs and receive notifications if they exceed a predefined threshold.

### Professional Exam Scenarios
#### Scenario 1
You are tasked with implementing a solution for managing patches across multiple accounts within an [[AWS Organization]]. Which of the following approaches would you recommend?

A) Implement [[ssm|SSM Patch Manager]] in one central account and configure cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.
B) Create separate [[ssm|SSM Patch Manager]] instances in each account and use [[organizations|AWS Organizations]] to manage them.
C) Configure a third-party patching tool integrated with AWS to handle patching across all accounts.
D) Set up [[cicd|AWS CodePipeline]] pipelines in each account to apply patches automatically.

Correct answer: A
Justification: Option A enables centralized patch management using [[ssm|SSM Patch Manager]] while providing cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Options B, C, and D do not address the need for centralized patch management across multiple accounts.

#### Scenario 2
As part of your company's [[appsync|security]] policy, you must enforce a strict patching schedule for all [[ec2]] instances deployed in different AWS Regions. How would you centrally manage this requirement without creating custom scripts?

A) Leverage [[ssm|SSM Patch Manager]] and define maintenance windows based on the required patching schedule.
B) Utilize [[config|AWS Config]] Rules to enforce patching schedules based on tags.
C) Implement AWS [[cloudwatch]] Events to trigger [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] that enforce patching schedules.
D) Employ [[step-functions|AWS Step Functions]] to manage patching schedules for [[ec2]] instances.

Correct answer: A
Justification: Option A allows you to leverage [[ssm|SSM Patch Manager]] and define maintenance windows based on the required patching schedule. Other options do not directly support enforcing patching schedules for [[ec2]] instances.