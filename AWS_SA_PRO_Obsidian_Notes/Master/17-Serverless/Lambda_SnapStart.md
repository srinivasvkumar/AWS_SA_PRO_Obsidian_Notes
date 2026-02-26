## [[RDS_Instance_Types|1. Advanced Architecture]]: [[lambda]] SnapStart [[RDS_Instance_Types|Internals]]

SnapStart is a feature of [[lambda|AWS Lambda]] that enables faster cold start times by reusing the execution context from previous invocations. It works by storing a snapshot of the function's state in between executions, allowing for quicker initialization when the function is triggered again. This can be particularly useful in serverless applications that require fast response times or have high request rates.

When a [[lambda]] function is configured with SnapStart, AWS automatically saves a snapshot of the function's execution context after each invocation. The next time the function is invoked, if the same version of the function is used and there are available snapshots, AWS will create a new instance of the function using the saved state. This process reduces the amount of time required to initialize the function, as many of the necessary dependencies and configurations are already loaded.

It's important to note that SnapStart has some [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] and requirements. For example, it only supports provisioned concurrency and requires at least one successful invocation before it can create a snapshot. Additionally, SnapStart snapshots are stored for a limited period of time and may be deleted if not used within a certain timeframe.

### [[RDS_Instance_Types|Global Scale Considerations]]

SnapStart can be used in conjunction with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]], such as Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] and [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]], to improve the performance of serverless applications at a global scale. By configuring these services to route traffic to the nearest [[lambda]] function, it is possible to reduce latency and improve the overall user experience. However, it is important to carefully consider the costs associated with these services and ensure that they are properly integrated with SnapStart.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]: When Not to Use [[lambda]] SnapStart

The following table compares [[lambda]] SnapStart with other serverless compute options, such as AWS [[Fargate]] and AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]], highlighting their strengths and weaknesses:

| Service | Cold Start Time | Provisioned Concurrency | Cost |
| --- | --- | --- | --- |
| [[lambda]] SnapStart | Fast (typically < 1s) | Supports | Pay-per-use, plus additional charges for storage |
| AWS [[Fargate]] | Slow (depends on container size and resources) | Supports | Pay-per-use, based on vCPU and memory usage |
| AWS [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]] | Fast (typically < 1s) | Does not support | Pay-per-use, based on number of active containers and runtime duration |

### Anti-Patterns

* Using SnapStart in situations where cold start times are not a significant concern
* Configuring SnapStart without considering the potential costs associated with storage and provisioned concurrency
* Relying solely on SnapStart for high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], without implementing additional redundancy measures

## [[RDS_Instance_Types|3. Security & Governance]]: Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Cross-Account Access

To implement SnapStart securely and govern its usage effectively, it is essential to define appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and cross-account access rules. Here are a few JSON policy snippets demonstrating how to grant permissions for SnapStart:

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy for [[lambda]] Invocation**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunction"
            ],
            "Resource": [
                "arn:aws:lambda:*:*:function:my-function*"
            ]
        }
    ]
}
```
**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy for Provisioned Concurrency**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:AddProvisionedConcurrencyConfig",
                "lambda:RemoveProvisionedConcurrencyConfig"
            ],
            "Resource": [
                "arn:aws:lambda:*:*:function:my-function*"
            ]
        }
    ]
}
```
**Cross-Account Access**

To allow cross-account access to a [[lambda]] function with SnapStart enabled, you need to modify the resource-based policy attached to the function. Here's an example policy statement:

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
                "lambda:InvokeFunction"
            ],
            "Resource": "arn:aws:lambda:*:*:function:my-function*"
        }
    ]
}
```
In addition to defining [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], [[organizations]] should also enforce [[appsync|security]] [[iam|best practices]] through Service Control [[policies]] (SCPs) and other governance mechanisms.

## [[RDS_Instance_Types|4. Performance & Reliability]]: Throttling Limits and Exponential Backoff Strategies

When using SnapStart, it is crucial to consider throttling limits and exponential backoff strategies to ensure high performance and reliability.

### Throttling Limits

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] with SnapStart have the same throttling limits as regular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. To avoid hitting these limits, developers should monitor the `InvocationRequestCountPerSecond` metric and adjust their provisioned concurrency settings accordingly.

### Exponential Backoff Strategies

Implementing exponential backoff strategies can help mitigate issues related to throttling and transient [[api-gateway|errors]]. When designing a serverless application with SnapStart, consider the following guidelines:

* Implement retry logic with increasing delays between retries (e.g., 2^n × 100ms, where n is the number of retries)
* Set a maximum number of retries before returning an error to the caller
* Use the `Status Code` and `Retry-After` headers in the [[lambda]] function response to determine whether a retry is necessary

## [[RDS_Instance_Types|5. Cost Optimization]]: Granular Cost Controls

SnapStart can increase the cost of running a [[lambda]] function due to the additional storage and provisioned concurrency requirements. To optimize costs, consider the following strategies:

* Enable SnapStart only for specific functions that require fast cold start times
* Monitor and delete unused SnapStart snapshots to avoid unnecessary storage costs
* Adjust provisioned concurrency settings based on the expected request rate and throttling limits

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1: Multi-Account Strategy

Suppose your organization uses multiple AWS accounts for development, testing, and production environments. How would you implement SnapStart across these accounts while ensuring proper access control and cost management?

#### Solution

To implement SnapStart in a multi-account environment, follow these steps:

1. Define [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] in each account to grant permission to invoke [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and manage provisioned concurrency.
2. Create a centralized [[organizations|AWS Organizations]] [[SCP]] to enforce cost management and limit the usage of SnapStart to specific functions.
3. Implement AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share SnapStart-enabled [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] between accounts.
4. Document and communicate the usage and cost implications of SnapStart to stakeholders.

#### Incorrect Answers

* Sharing the [[lambda]] function code directly between accounts, without using [[ram]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, could lead to unintended access and violate [[appsync|security]] [[iam|best practices]].
* Allowing unrestricted access to all [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] without applying any [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] could result in unexpected costs and [[appsync|security]] vulnerabilities.

### Scenario 2: High Availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]]

Your company relies on a serverless application that processes real-time financial transactions. The application consists of several [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] connected by Amazon [[eventbridge]] and Amazon [[dynamodb]] Streams. How would you design a highly available and fault-tolerant architecture that leverages SnapStart to minimize cold start times?

#### Solution

To build a highly available and fault-tolerant architecture with SnapStart, follow these steps:

1. Enable SnapStart for critical [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] that require fast cold start times.
2. Implement provisioned concurrency for these functions based on the expected request rate and throttling limits.
3. Distribute the workload across multiple Availability Zones (AZs) using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] to load balance requests and minimize the impact of AZ failures.
4. Implement automatic failover and backup using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] and Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for data persistence.
5. Monitor the system using AWS [[cloudwatch]] and set up alarms to notify you of any issues.

#### Incorrect Answers

* Assuming that SnapStart alone provides sufficient high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] could lead to unplanned downtime and data loss.
* Ignoring the distribution of workload across multiple AZs could result in higher latency and reduced availability during AZ failures.