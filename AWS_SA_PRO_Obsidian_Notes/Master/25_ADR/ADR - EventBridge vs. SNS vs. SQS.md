```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
priority: high
```

## Enhanced Technical [[vpc-flow-logs|Notes]]: Amazon [[eventbridge]] vs. Amazon [[sns]] vs. Amazon [[sqs]]

### 1. UNDER-THE-HOOD MECHANICS

#### Amazon [[eventbridge]]
- **Internal Data Flow**: [[eventbridge]] receives events from various sources and routes them to targets based on rules.
- **Consistency Model**: Eventually consistent; ensures all events are delivered but might not be immediately visible across the system.
- **Underlying Infrastructure Logic**: Uses a rule-based engine for event filtering and routing. Supports schema validation with Schema Registry.

#### Amazon [[sns]]
- **Internal Data Flow**: Publishes messages to topics, which subscribers can then consume.
- **Consistency Model**: Eventually consistent; ensures message delivery but might not be immediate across all endpoints.
- **Underlying Infrastructure Logic**: Topics are centralized publish points that distribute messages to multiple endpoints (e.g., [[sqs]] queues, HTTP endpoints).

#### Amazon [[sqs]]
- **Internal Data Flow**: Queues store and forward messages from producers to consumers in a first-in-first-out order.
- **Consistency Model**: Eventually consistent; ensures message delivery but might not be immediately visible across the system.
- **Underlying Infrastructure Logic**: Supports FIFO (First-In-First-Out) queues for ordered processing, as well as standard queues for high throughput.

### 2. EXHAUSTIVE FEATURE LIST

#### Amazon [[eventbridge]]
- **Event Sources and Targets**: Integration with AWS services, custom applications via [[api-gateway|API Gateway]] or [[lambda]].
- **Rules Engine**: Filters events based on [[cloudformation|conditions]] before routing to targets ([[lambda]], [[sns]], [[sqs]]).
- **Schema Registry**: Validates event schemas using JSON schema definitions.
- **[[cloudwatch]] Metrics**: Monitors [[eventbridge]] activity for operational visibility.

#### Amazon [[sns]]
- **Topic Subscription**: Supports multiple protocols (e.g., HTTP, HTTPS, email, SMS, [[sqs]]).
- **Message Filtering**: Filters messages based on message attributes before delivering to subscribers.
- **Fan-Out Architecture**: Distributes a single published message to all subscribed endpoints.
- **Dead Letter Queues (DLQs)**: Routes failed deliveries to DLQ for further processing.

#### Amazon [[sqs]]
- **Queue Types**: Standard queues for high throughput and FIFO queues for ordered messages.
- **Visibility Timeout**: Controls how long a message is hidden from other consumers after it has been received.
- **Message Attributes**: Metadata about the message that can be used for filtering or additional context.
- **DLQs**: Automatically routes failed messages to DLQ.

### 3. STEP-BY-STEP IMPLEMENTATION

#### Amazon [[eventbridge]]
1. Create an event bus (default is available).
2. Define rules with patterns to match specific events.
3. Configure targets ([[lambda]], [[sns]], [[sqs]]) for each rule.
4. Test rules using [[cloudwatch]] Events and check target logs.

#### Amazon [[sns]]
1. Create a topic.
2. Subscribe endpoints to the topic (e.g., [[sqs]] queue, HTTP endpoint).
3. Publish messages to the topic.
4. Monitor delivery status via Amazon [[sns]] console or API.

#### Amazon [[sqs]]
1. Create a queue (standard/FIFO).
2. Configure attributes like message [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] and visibility timeout.
3. Send messages to the queue using SDKs or CLI.
4. Consume messages from the queue using consumers.

### 4. TECHNICAL LIMITS

- **[[eventbridge]]**: Maximum of 50 rules per event bus, adjustable up to 100 via support request (as of 2026).
- **[[sns]]**: Up to 1 million messages per second in one topic; 10,000 topics per account.
- **[[sqs]]**: Standard queue: 340,000 requests per second; FIFO: 3,000 requests per second.

### 5. CLI & API GROUNDING

#### Amazon [[eventbridge]]
```bash
aws events put-rule --name "MyRule" --event-pattern '{"source": ["aws.ec2"]}'
aws lambda create-event-source-mapping --function-name MyFunction --batch-size 10 --enabled --event-source-arn arn:aws:events:us-east-1:123456789012:eventbus/default
```

#### Amazon [[sns]]
```bash
aws sns create-topic --name myTopic
aws sns subscribe --topic-arn arn:aws:sns:us-west-2:123456789012:myTopic --protocol email --notification-endpoint user@example.com
aws sns publish --topic-arn arn:aws:sns:us-west-2:123456789012:myTopic --message "Hello World"
```

#### Amazon [[sqs]]
```bash
aws sqs create-queue --queue-name myQueue
aws sqs send-message --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/myQueue --message-body "test message"
aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/myQueue
```

### 6. PRICING & TCO

#### Amazon [[eventbridge]]: Charges for rules and event processing; free tier includes 1M rule evaluations per month.
#### Amazon [[sns]]: Free up to 1 million messages per month, then $0.50 per 1M additional messages.
#### Amazon [[sqs]]: Standard: $0.43 per 1M requests (first 1M free). FIFO: $0.58 per 1M requests (first 1M free).

### 7. COMPETITOR COMPARISON

- **[[eventbridge]] vs. SNS/SQS**: [[eventbridge]] provides a more event-driven architecture with advanced filtering and routing capabilities, while [[sns]] and [[sqs]] are traditional message brokers.
- **Architectural Trade-offs**:
    - **Operational Excellence**: [[eventbridge]] is better for complex rule-based event processing; SNS/SQS offers simpler but robust fan-out architectures.
    - **[[00_Master_Links_and_Intro|Cost Optimization]]**: Use [[eventbridge]] for high-frequency events with advanced filtering, use SNS/SQS for cost-effective message distribution.

### 8. CONTEXTUALIZATION WITHIN THE AWS Ecosystem

[[eventbridge]] integrates with [[cloudwatch]], [[lambda]], and [[api-gateway|API Gateway]]. [[sns]] and [[sqs]] integrate closely with each other and with services like [[lambda]] for event processing.

### 9. REAL-WORLD USE CASES & EXAMPLES

#### Amazon [[eventbridge]]
- **Use Case**: Event-driven microservices architecture where events trigger specific [[00_Master_Links_and_Intro|Lambda functions]] based on event type using [[eventbridge]].

#### Amazon SNS/SQS
- **Use Case**: Decoupled systems architecture, such as notification fan-out or message queuing in a distributed application.

### 10. ARCHITECTURAL DIAGRAMS

Include diagrams showing how each service integrates with [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]] (e.g., [[eventbridge]] triggering [[lambda]], [[sns]] distributing to multiple [[sqs]] queues).

### 11. EXAM TRAPS & COMMON MISCONCEPTIONS

- **Amazon [[eventbridge]]**: Misunderstanding the difference between rule patterns and event structures.
- **Amazon SNS/SQS**: Confusing FIFO vs. standard queue features.

### 12. DECISION MATRIX FORMAT

| Feature          | Amazon [[eventbridge]]                          | Amazon [[sns]]                                      | Amazon [[sqs]]                                       |
|------------------|--------------------------------------|------------------------------------------|-------------------------------------------|
| Data Flow        | Rule-based event routing             | Publish-subscribe model                  | Queue-based message passing               |
| Use Case         | Complex event-driven workflows       | Fan-out architecture                     | Decoupled system communication            |
| Exam Focus       | Event processing and [[lambda]] triggers | Message distribution to multiple targets | Reliable message storage and ordering     |

### 13. LATEST AWS UPDATES (2026)

- **Amazon [[eventbridge]]**: Enhanced schema validation support.
- **Amazon SNS/SQS**: New features for dead-letter queues.

### 14. EXAM QUESTIONS & SCENARIOS

**Scenario**: Design a system where AWS [[00_Master_Links_and_Intro|CloudTrail]] events trigger specific [[00_Master_Links_and_Intro|Lambda functions]] based on event type using Amazon [[eventbridge]].

### 15. INTEGRATION WITH [[00_Master_Links_and_Intro|OTHER AWS SERVICES]]

- **Amazon [[eventbridge]]**: Integrates with [[cloudwatch]] for monitoring, and SNS/SQS for downstream processing.
- **Amazon SNS/SQS**: Interoperable with services like [[dynamodb]] Streams, [[lambda]] for scalable message processing.

### 16. RELEVANCE TO SAP-C02 EXAM

Understanding the differences in event handling, message distribution, and queue management is critical for designing scalable and resilient architectures on AWS.

## Enterprise Governance Layers
- **[[organizations|AWS Organizations]]**: Manage multiple accounts within a single organization.
- **Service Control [[policies]] (SCPs)**: Define allowed actions across member accounts.
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Share resources across accounts securely.
- **[[00_Master_Links_and_Intro|CloudTrail]] Auditing**: Monitor API calls and resource changes for compliance.

## Troubleshooting Complex Failure Modes

### [[kms]] Key Policy Conflicts in Cross-Account Migrations
- **Scenario 1**: A key policy conflict occurs when migrating [[sns]] topics with cross-account subscriptions using KMS-managed keys.
    - **Troubleshoot**: Check if the [[kms]] key [[policies]] allow the necessary permissions for cross-account access. Ensure that the master and replica keys have compatible [[policies]].

### Operational Excellence vs. [[00_Master_Links_and_Intro|Cost Optimization]]
- **Amazon [[eventbridge]]**: Use [[eventbridge]] for high-frequency events with advanced filtering, ensuring operational excellence in complex workflows.
- **Amazon SNS/SQS**: Use [[sns]] and [[sqs]] for cost-effective message distribution and fan-out architectures, focusing on [[00_Master_Links_and_Intro|cost optimization]] without sacrificing reliability.

## Conclusion

This overview provides a comprehensive breakdown of Amazon [[eventbridge]], Amazon [[sns]], and Amazon [[sqs]], including their underlying mechanics, features, implementation steps, pricing models, competitor comparisons, and integration within the broader AWS ecosystem. It also includes enterprise governance layers, troubleshooting complex failure modes, architectural trade-offs, and exam traps to equip SAP-C02 candidates with detailed knowledge necessary for exam success.

### Related [[vpc-flow-logs|Notes]]

- [[Amazon EventBridge]]
- [[Amazon SNS]]
- [[Amazon SQS]]
```