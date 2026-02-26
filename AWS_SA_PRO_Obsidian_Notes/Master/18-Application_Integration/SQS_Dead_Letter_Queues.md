**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[sqs]] Dead Letter Queues (DLQs) are separate [[sqs]] queues that other [[sqs]] queues can target when they encounter specific error [[cloudformation|conditions]] or message processing failures. A DLQ is critical in helping developers diagnose and address issues within their applications.

Internally, an [[sqs]] queue configured with a DLQ will send messages that exceed a predefined number of receives or visibility timeout without being processed to the designated DLQ after configuring maxReceiveCount or visibilityTimeoutNotificationRoute.

For global scale architecture, using Amazon [[sqs]] FIFO (First-In-First-Out) queues with supported services like [[sns]], [[lambda]], and [[kinesis|Kinesis Data Firehose]] enables cross-region replication. However, [[sqs]] does not natively support active/active DLQ configurations across regions. To implement a multi-region setup, you would need to create a custom solution using [[cloudwatch]] Events, [[lambda]], and [[sns]].

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Cases | Anti-patterns |
| --- | --- | --- |
| [[sqs]] | Decouple application components, process asynchronous tasks, scalable messaging. | Not suitable for real-time messaging requirements due to its polling model. |
| [[kinesis|Kinesis Data Streams]] | Real-time data streaming, sequential processing, analytics, machine learning. | Overkill for simple task queues and decoupling use cases. |
| [[eventbridge]] | Serverless event routing, schedule-based invocations, cross-service choreography. | Shouldn't be used for general-purpose message queuing or task processing. |

Common anti-patterns include:

- Using [[sqs]] DLQs for real-time monitoring or alerting purposes instead of implementing proper monitoring solutions.
- Inappropriately setting `maxReceiveCount` or `visibilityTimeoutNotificationRoute`, causing legitimate [[api-gateway|errors]] to be sent to the DLQ.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow the principle of least privilege. Here's an example JSON policy snippet allowing an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user to manage only one specific DLQ:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:GetQueueAttributes",
                "sqs:PurgeQueue"
            ],
            "Resource": [
                "arn:aws:sqs:*:123456789012:MyDeadLetterQueue"
            ]
        }
    ]
}
```

Cross-account access can be granted by modifying the resource portion of the above example to include the appropriate account ID. For centralized governance, [[organizations]] can enforce specific [[sqs]] permissions through Service Control [[policies]] (SCPs).

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the queue type and region. For standard queues, the maximum receive rate is 10 messages per second. The limit increases up to 30,000 messages per second with ordered attributes disabled.

Exponential backoff strategies help prevent applications from repeatedly failing to process messages. An example strategy includes increasing the delay between retries exponentially while capping the retry attempts based on business needs.

HA/DR patterns involve creating backup queues in different availability zones or regions. Implementing a custom [[dr]] solution requires additional development efforts and maintenance costs compared to native features.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved by applying tags to your resources and monitoring usage through [[billing|Cost Explorer]]. Calculating the cost of sending messages to a DLQ depends on factors such as the number of messages, API calls, and data transfer.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

*Scenario 1*: A company has multiple accounts and wants to centrally manage DLQs for better control and monitoring. How would you achieve this?

Answer: Implement a centralized [[vpc-flow-logs|logging]] and monitoring system using [[organizations|AWS Organizations]], [[cloudwatch|CloudWatch Logs]], and Metrics. Create a service role in each member account allowing [[organizations]] to push events to the master account. Apply SCPs to restrict DLQ management to relevant users or groups.

*Scenario 2*: A high-traffic e-commerce platform uses [[sqs]] for order processing but encounters throttling issues. What steps can be taken to mitigate these issues?

Answer: Increase the default DLQ limits by submitting an AWS Support case. Enable auto-scaling for the [[ec2]] instances handling the order processing workload. Implement a load balancing mechanism like [[sqs]] Message Groups to distribute traffic among multiple consumers more efficiently. Adjust the `maxReceiveCount` parameter to balance between reliability and performance.