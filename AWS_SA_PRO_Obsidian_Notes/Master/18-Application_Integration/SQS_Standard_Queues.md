## [[RDS_Instance_Types|1. Advanced Architecture]]
[[sqs]] Standard Queues provide best-effort ordering and at-least-once delivery guarantees. Understanding their [[RDS_Instance_Types|internals]] is crucial when designing scalable and performant systems.

### Message Visibility Timeout
When a consumer receives a message from an [[sqs]] queue, it becomes invisible to other consumers for a configurable duration called the *message visibility timeout*. This mechanism prevents two consumers from processing the same message concurrently. If the consumer doesn't delete the message within the visibility timeout period, it becomes visible again and can be reprocessed by another consumer.

### Dead Letter Queues (DLQ)
To handle failed message processing, [[sqs]] supports DLQs that act as secondary queues to store messages that couldn’t be processed successfully after a maximum number of retries. Monitoring and alerting on DLQ metrics like `approximatemessagesdelayed` and `approximatenumberofmessagesvisible` help in identifying potential issues early.

### Global Scaling Considerations
For global applications requiring geographically dispersed services, FIFO and Standard queues support active-active replication using Amazon [[sqs]] Active [[mq]]. The solution involves deploying separate [[sqs]] resources per region and implementing custom logic to maintain consistency between regions.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]
Here are some scenarios when you should avoid using [[sqs]] Standard Queues and possible alternatives:

| Use Case | Anti-Pattern | Alternative |
|---|---|---|
| Ordered Messaging | Using Standard Queues without FIFO support | [[kinesis|Kinesis Data Streams]] or [[sqs]] FIFO Queues |
| At-most-once Delivery Guarantee | Using Standard Queues | [[kinesis|Kinesis Data Streams]] or [[dynamodb]] Streams |
| Low Latency Requirements | Using [[sqs]] due to higher latencies | [[kinesis|Kinesis Data Streams]] or Direct Connect |
| Cost-sensitive Applications | High messaging costs | [[eventbridge]], [[sns]] Topics, or self-hosted messaging solutions |

## [[RDS_Instance_Types|3. Security & Governance]]
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and organization SCPs play essential roles in securing your [[sqs]] infrastructure.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]
Use JSON [[policies]] to allow or deny specific actions based on resource attributes. For example, granting permissions to send messages to a queue while restricting delete permissions:
```json
{
    "Effect": "Allow",
    "Action": ["sqs:SendMessage"],
    "Resource": "arn:aws:sqs:*:123456789012:MyQueue"
},
{
    "Effect": "Deny",
    "Action": ["sqs:DeleteQueue"],
    "Resource": "arn:aws:sqs:*:123456789012:MyQueue"
}
```
### Cross-Account Access
Cross-account access requires specifying the source account ARN in the resource-based policy attached to the [[sqs]] queue. Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:root"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:us-east-1:123456789012:MyQueue",
      "Condition": {
        "ArnLike": {
          "aws:SourceVpc": "arn:aws:ec2:us-east-1:111122223333:vpc/*"
        }
      }
    }
  ]
}
```
### Organization Service Control [[policies]] (SCPs)
Organization SCPs limit the usage of [[sqs]] across member accounts. For instance, denying creation of [[sqs]] resources:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCreateSQS",
      "Effect": "Deny",
      "Action": [
        "sqs:CreateQueue"
      ],
      "Resource": "*"
    }
  ]
}
```
## [[RDS_Instance_Types|4. Performance & Reliability]]
Throttling limits, exponential backoff strategies, and HA/DR patterns are critical aspects of ensuring optimal performance and reliability.

### Throttling Limits
Standard queues have an initial quota of up to 120,000 incoming API requests per second and 10,000 outgoing API requests per second. To increase these limits, submit a service quotas request.

### Exponential Backoff Strategies
Exponential backoff strategies prevent your application from overwhelming the [[sqs]] service during temporary [[api-gateway|errors]]. Implement them using the Jitter technique to randomize the time intervals between retries.

### HA/DR Patterns
Implement highly available and disaster-resilient architecture by distributing [[sqs]] resources across multiple AZs and replicating them in different regions. Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and failover mechanisms to route traffic to healthy endpoints.

## [[RDS_Instance_Types|5. Cost Optimization]]
Granular cost controls include monitoring usage trends, setting alarms, and applying tags to allocate expenses accurately. Calculate [[sqs]] costs using the following formula:
```less
MonthlyCost = (NumberOfMessages * PricePerRequest) + (NumberOfBytes * PricePerGB)
```
## 6. Professional Exam Scenario
### Question 1
Suppose you must implement a system that processes financial transactions with guaranteed at-least-once delivery and low-latency requirements. Which service(s) would you choose? Explain your answer.

Correct Answer: Since financial transactions require low-latency and at-least-once delivery guarantees, neither [[sqs]] Standard nor FIFO queues meet these requirements. Instead, consider using [[kinesis|Kinesis Data Streams]] or [[dynamodb]] Streams, which offer lower latencies and at-most-once delivery semantics.

Incorrect Answer: Using [[sqs]] Standard Queues might not ensure low-latency requirements. Although [[sqs]] FIFO Queues guarantee FIFO order, they may still introduce unnecessary latencies compared to [[kinesis|Kinesis Data Streams]] or [[dynamodb]] Streams.

### Question 2
Consider a multi-account environment with several AWS accounts containing shared [[sqs]] resources. How would you secure these resources?

Correct Answer: In a multi-account environment, you can apply [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access rules, and organization SCPs to secure [[sqs]] resources. Specifically, create resource-based [[policies]] allowing only specific actions from particular principal entities, utilize cross-account access configurations, and define organization SCPs limiting the usage of [[sqs]] resources.

Incorrect Answer: Relying solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] within individual AWS accounts does not sufficiently address [[appsync|security]] concerns since those [[policies]] do not extend beyond the scope of each account. Hence, cross-account access configurations and organization SCPs are necessary components of a comprehensive [[appsync|security]] strategy.