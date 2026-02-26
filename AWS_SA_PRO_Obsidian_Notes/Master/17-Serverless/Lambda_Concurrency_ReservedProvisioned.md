**[[RDS_Instance_Types|1. Advanced Architecture]]**

At the core of [[lambda]] is its ability to run code without managing servers. AWS handles capacity management, patching, and scaling so that you can focus on your code. [[lambda]] executes your code in response to events and manages the underlying compute resources for you.

Internally, [[lambda]] operates as follows:

* Containers: Your function code, along with the associated dependencies and environment variables, is packaged into a container before it is executed. This container is managed by [[lambda]], not visible or accessible to customers.
* Execution Environment: Each [[lambda]] function execution runs in its own isolated execution environment. The [[ram]] allocated to your function configuration determines the size of the execution environment.
* Concurrency: By default, [[lambda]] manages concurrent executions for you within a single account based on the region’s provisioned concurrency limit.

[[RDS_Instance_Types|Global Scale Considerations]]:

* Regions & AWS Resource Access: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] operate in a single region and cannot directly access resources in other regions. To overcome this limitation, use an [[api-gateway|API Gateway]] REST API with regional endpoints in multiple regions connected to the same [[lambda]] function in the primary region via AWS Backend Services (HTTP(S) Integration).
* [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Configuration & Scaling: When using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] configurations, [[lambda]] creates ENIs in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and manages the availability of these ENIs. If your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] has insufficient IP addresses available, [[lambda]] might not be able to create the necessary ENIs, causing the function to scale slowly or fail to scale at all.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Stateless Computing | Scaling Model | Cost Model |
| --- | --- | --- | --- |
| [[lambda]] | Yes | Event-driven | Pay-per-use |
| [[Fargate]] | Yes | Manual | Pay-per-use |
| [[ecs]] | No | User-managed | Pay-per-use |

Anti-pattern: Using [[lambda]] for long-running processes (e.g., > 15 minutes). Instead, consider using [[ecs]] or [[Fargate]].

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "lambda:CreateFunction",
              "lambda:UpdateFunctionCode"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-Account Access:

* [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Role: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to invoke the [[lambda]] function. Attach the role to a user or group.
* [[lambda]] Permissions: In the destination account, add a permission to allow cross-account invocation.

Organization SCPs:

* Centralized Management: Implement Organization Service Control [[policies]] (SCPs) to restrict actions like creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] outside specific accounts.
* Least Privilege: Ensure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles have the minimum required permissions for each resource.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:

* Per-region throttle limits apply to the number of concurrent requests for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].
* Increase the limit by submitting a service limit increase through the Support Center.

Exponential Backoff Strategies:

* Handle [[api-gateway|errors]] gracefully by implementing retry logic with exponential backoff and jitter.
* Apply this strategy when connecting to databases, message queues, or other services from [[lambda]].

HA/DR Patterns:

* Deploy stateless applications across multiple regions to provide high availability.
* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover for automatic failover between regions.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular Cost Controls:

* Monitor costs using [[cloudwatch|CloudWatch Alarms]] and [[billing]] Alerts.
* Enable Provisioned Concurrency if predictable latency is required.

Calculation Examples:

* Calculate the cost of running a [[lambda]] function with 1M invocations per month, 1GB of data transferred, and 500ms average duration with 128MB memory.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A: A company needs to process large files in batches while maintaining low latency. They currently use [[lambda]] but experience throttling issues due to exceeding maximum concurrency limits. Propose a solution that eliminates throttling while ensuring minimal latency.

Correct Answer: Introduce Provisioned Concurrency for the [[lambda]] function to ensure there are always pre-warmed instances ready to serve incoming requests.

Incorrect Answers:

* Migrating to [[ecs]]: While [[ecs]] provides more control over scaling, it does not inherently address [[lambda]] throttling [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]].
* Adding more parallel processing steps: This would not necessarily eliminate throttling since the issue lies in the number of concurrent requests.

Scenario B: A media organization wants to store photos in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and transcode them upon upload. However, they don't want to pay for idle compute resources during periods of low activity. What architecture should they adopt?

Correct Answer: Use an event-driven architecture with [[lambda]] triggered by [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] object creation events.

Incorrect Answers:

* Running [[ec2]] instances continuously: This approach results in unnecessary costs during periods of low activity.
* Storing images in [[dynamodb]]: [[dynamodb]] is not suitable for storing binary objects like images efficiently.