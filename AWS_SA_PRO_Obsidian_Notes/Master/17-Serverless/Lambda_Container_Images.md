## [[RDS_Instance_Types|1. Advanced Architecture]]: [[lambda|Lambda Container Images]]

### [[RDS_Instance_Types|Internals]]
[[lambda|AWS Lambda]] allows you to run your code without provisioning or managing servers. With container images, you can bring your own environment and dependencies, including libraries, tools, and data files. The container image includes everything that is required to run your code.

When using container images in [[lambda]], the invocation process involves four steps:

1. Pulling the image from Amazon Elastic Container Registry (ECR) or another compatible registry.
2. Starting the container with the pulled image.
3. Running user initialization inside the container.
4. Executing the function handler within the container.

### [[RDS_Instance_Types|Global Scale Considerations]]
Lambdas with container images do not currently support global scaling. However, you can build solutions using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] and ECR repositories in multiple regions to achieve similar results. This approach requires careful management of synchronization between different deployments and understanding the potential impact on performance and costs.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Feature                     | [[lambda]] Container Image          | Alternatives               |
|-----------------------------|----------------------------------|----------------------------|
| Language Support            | Broad but depends on base image  | Narrower native support    |
| Initialization Time         | Slower due to container startup   | Faster                     |
| Dependency Management        | Robust dependency management      | Limited built-in support    |
| Resource Utilization        | Higher resource utilization       | Lower resource utilization   |
| Cold Start Performance       | Worse than non-container variants | Better                     |

Anti-pattern example: Using container images when the same functionality can be achieved with less complexity using native [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:GetLogStream*,lambda:GetLogEvents"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```
Cross-account access:

* Grant permissions to the source account at the destination account level.
* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with cross-account access.
* Attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy allowing necessary actions to the role.

Organization Service Control [[policies]] (SCPs):

* Implement centralized control over [[lambda]] resources by applying SCPs at the organization level.
* Prevent unwanted changes to resources by restricting specific API calls.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits:

* [[lambda]] imposes throttling limits based on concurrent executions.
* To mitigate throttling issues, implement exponential backoff strategies.

High Availability / [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]] (HA/DR) patterns:

* Distribute workloads across multiple availability zones.
* Implement failover mechanisms using [[route53]] [[route53|health checks]] and [[dynamodb]] Streams.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls:

* Monitor usage and set alarms based on [[cloudwatch]] metrics.
* Enable Provisioned Concurrency for predictable workloads.
* Use [[lambda]] Data Transfer pricing to optimize data transfer costs.

Calculation example:

Suppose you have a container image [[lambda]] running in us-east-1 with 50ms execution time, 100 MB memory, and 1 million requests per month. The monthly cost would be approximately $0.95, calculated as follows:

* Compute cost: $0.0000006144 per request * 1M requests = $0.0006144
* Memory cost: ($0.000000000104 \* 100 MB \* 1 million requests) + $0.000000000000000804 = $0.07
* Total compute and memory cost: $0.0006144 + $0.07 = $0.0706144
* Data transfer cost: Assume 1 KB outbound per request, so 1GB total. In us-east-1, first 1 GB is free, so no data transfer cost.
* Therefore, the total cost is $0.0706144.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

Scenario 1:
You need to deploy a serverless application with a custom runtime written in Rust. The solution must handle high load and provide a robust deployment mechanism. Which architecture should you choose?

Correct answer: Deploy the Rust application as a container image in [[lambda]]. Use ECR as the container registry and ensure the application has proper resource configuration and error handling.

Incorrect answer: Deploy the Rust application as a standalone [[lambda]] function. Standalone [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] require more extensive setup and configuration for custom runtimes, which may increase complexity and maintenance efforts.

Scenario 2:
Your company operates in multiple AWS accounts and needs to enforce a consistent naming convention for all [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. How can you accomplish this while minimizing operational overhead?

Correct answer: Apply an Organizational Service Control Policy ([[SCP]]) that enforces the desired naming convention. The [[SCP]] will prevent renaming functions outside the allowed pattern.

Incorrect answer: Implement a custom [[lambda]] function responsible for validating function names. While this solution provides some control, it increases operational overhead and does not guarantee compliance across all accounts.