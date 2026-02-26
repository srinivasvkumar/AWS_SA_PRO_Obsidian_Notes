Advanced Architecture
---------------------

At its core, [[lambda]] triggers enable the execution of serverless functions in response to specific events. These events can originate from various AWS services such as [[api-gateway|API Gateway]], [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets, or [[cloudwatch|CloudWatch Logs]]. To ensure scalability and high availability, [[lambda]] leverages a multi-region, multi-AZ infrastructure that automatically distributes incoming requests across multiple instances.

### [[RDS_Instance_Types|Global Scale Considerations]]

When building global applications, it is essential to understand how [[lambda]] handles cross-region invocations. In such cases, [[lambda]] uses AWS's private network ([[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]]) to route traffic between regions while preserving the client's IP address. This allows developers to build centralized [[vpc-flow-logs|logging]] and monitoring systems without relying on complex networking configurations.

### Under the Hood Mechanics

[[lambda]] relies on container technology to manage function executions efficiently. Upon an event trigger, [[lambda]] packages the function code along with its dependencies into a container and assigns it to a worker process within one of the available Availability Zones (AZs). To optimize resource usage further, [[lambda]] employs a concept called "provisioned concurrency" which keeps some containers pre-initialized, thereby reducing cold start times significantly.

Comparison & Anti-Patterns
---------------------------

| Service           | Use Case                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| [[lambda]]            | Real-time file processing, web application backends, [[iot]] device communication |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]]    | Orchestrating complex workflows, managing stateful processes             |
| Batch            | High-throughput data processing, large-scale ETL operations              |
| [[Fargate]] ([[ecs]])     | Containerized microservices, custom runtime environments                  |

Anti-patterns include using [[lambda]] for long-running tasks (e.g., more than 15 minutes) or when tight control over underlying resources is necessary.

[[appsync|Security]] & Governance
----------------------

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] often involve nested [[cloudformation|conditions]], multi-service roles, and principal-based permissions. Here's an example of granting an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user permission to invoke a [[lambda]] function:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:*:123456789012:function:my-function",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpc": "vpci-1a2b3c4d"
        }
      }
    }
  ]
}
```

Cross-account access requires configuring appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships. For instance, to allow a user from account A to invoke a [[lambda]] function in account B, you need to create a role in account B with the necessary permissions and then add a trust policy allowing users from account A to perform the `sts:AssumeRole` action.

Organization Service Control [[policies]] (SCPs) provide centralized governance by limiting actions at the organization level. For example, setting up an [[SCP]] restricting certain [[lambda]] actions could prevent unauthorized creation or modification of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] across all accounts within the organization.

Performance & Reliability
--------------------------

To avoid exceeding throttling limits, consider implementing exponential backoff strategies when handling failures. By default, [[lambda]] allows up to 1,000 concurrent executions per region. However, this limit can be increased upon request.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns typically involve deploying [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] across multiple regions and linking them with Amazon Global Accelerator. This setup ensures minimal downtime during outages and enables seamless failover between regions.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
-----------------

Granular cost controls include enabling provisioned concurrency for critical functions and adjusting memory settings based on actual requirements. The following formula calculates the cost of running a [[lambda]] function:

Cost = (Duration \* Requests) \* Compute Price + (Requests \* Request Price)

For example, if a [[lambda]] function runs for 500ms, has 10,000 invocations, and uses 512 MB of [[ram]]:

Cost = (0.5s \* 10,000) \* $0.000000401 + (10,000 \* $0.00000020) = $0.00803

Professional Exam Scenarios
---------------------------

Scenario 1: Your company wants to implement real-time image processing for their e-commerce platform using [[lambda]]. They expect high traffic during holiday seasons and anticipate millions of images being uploaded daily.

Correct Answer: Implement a distributed architecture with multiple [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] triggered by [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket notifications. Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] for global load balancing and automatic failover. Set up provisioned concurrency to reduce cold start times. Monitor costs closely and adjust scaling settings accordingly.

Incorrect Answer: Rely solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|step functions]] to orchestrate the entire process since they may introduce additional latency due to their asynchronous nature.

Scenario 2: A media streaming company needs to transcode video files in near real-time using [[lambda]]. Each transcoding job takes approximately 15 minutes to complete.

Correct Answer: Use [[aws-batch|AWS Batch]] instead of [[lambda]] due to its support for long-running tasks and higher throughput capabilities.

Incorrect Answer: Increase the [[lambda]] timeout limit beyond the default 15 minutes, which is against [[iam|best practices]] and may lead to unpredictable performance issues.