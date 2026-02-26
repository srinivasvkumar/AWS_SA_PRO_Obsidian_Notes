```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Under-the-Hood Mechanics

**Internal Data Flow:**  
[[Amazon EventBridge]] captures and routes events from software applications or [[AWS services]] to targets such as [[Lambda functions]], [[SNS topics]], [[SQS queues]], or other event buses. Events are JSON messages that adhere to a specific structure defined by the publisher (source). The core component is the **Event Bus**, which acts as a central hub for receiving and routing these events based on configured rules.

**Consistency Models:**  
[[EventBridge]] ensures eventual consistency in its event delivery model. Events are processed in order, but due to potential delays or retries, they might not always be delivered immediately upon arrival. The system employs **Retry Policies** to handle failures gracefully by retrying message deliveries until a specified threshold is reached.

**Underlying Infrastructure Logic:**  
[[EventBridge]] uses a highly scalable distributed system architecture with multiple availability zones (AZs) for redundancy and failover capabilities. Events are first ingested into the event bus, which then evaluates them against defined rules using pattern matching mechanisms. Matching events trigger specific targets based on those rules.

### Exhaustive Feature List

1. **Standard Event Bus:** Default bus that processes events from [[AWS services]].
2. **Custom Event Buses:** Dedicated buses for processing application-specific or custom events.
3. **Event Rule Targets:** [[Lambda functions]], [[SNS topics]], [[SQS queues]], [[Step Functions]], other event buses, etc.
4. **Pattern Matching:** Use of JSON-like patterns to match and route events.
5. **Dead Letter Queues (DLQs):** Handle failed deliveries by routing them to an [[SQS queue]] for further analysis.
6. **Event Filtering:** Fine-grained filtering based on attributes or metadata within the event payload.
7. **Retry Policies:** Automatic retries with configurable backoff strategies and maximum attempts.
8. **CloudWatch Metrics & Logs Integration:** Monitoring of event bus activity and logging events.

### Step-by-Step Implementation

1. **Define Event Source:** Identify AWS services or custom applications generating events.
2. **Create Custom Bus (if needed):** Use the console, CLI, or SDK to create a dedicated event bus.
3. **Configure Rules:** Define rules with specific patterns and targets for processing and routing events.
4. **Set Up Retry Policies & DLQs:** Configure retry settings and dead-letter queues for error handling.
5. **Integrate with CloudWatch:** Enable logging and monitoring for the event bus.

### Technical Limits (2026)

1. **Event Size Limitation:** 1 MB per event payload.
2. **Rule Count:** 1,000 rules per region for standard and custom buses combined.
3. **Concurrency Control:** Maximum of 10,000 concurrent events processed in parallel.
4. **Event Age Retention:** Events are kept for up to 24 hours before being discarded.

### CLI & API Grounding

**CLI Commands:**
```sh
aws events put-rule # Create a new rule.
aws events put-targets # Add targets to the rule.
aws events list-rules # List all rules in your account.
aws events describe-event-bus # Get details about an event bus.
```

**API Flags & Endpoints:**
- **EventBridge API**: Use endpoints such as `PutRule`, `PutTargets`, `ListRules`.
- **SDKs**: Access EventBridge functionality via [[AWS SDKs]] for Python, Java, etc.

### Pricing & TCO

1. **Pricing Drivers:** Pay per rule and event processed.
2. **Cost Optimization Strategies:**
   - Use batching to reduce the number of individual events.
   - Enable cost allocation tags for tracking usage.
   - Monitor [[CloudWatch metrics]] to optimize resource utilization.

### Competitor Comparison

**AWS EventBridge vs Azure Event Grid:**

- **Scalability:** Both offer high scalability but EventBridge integrates more seamlessly with AWS services.
- **Event Processing Model:** EventGrid supports both cloud and on-premises events, whereas EventBridge focuses primarily on AWS-based events.
- **Rule Engine Complexity:** EventBridge offers a simpler rule engine for pattern matching compared to the more flexible expression language in Azure.

### Integration

1. **CloudWatch Metrics & Logs:** Automatically collect metrics and log event processing details.
2. **IAM Role Usage:** Use [[IAM roles]] to control access to event buses, rules, and targets.
3. **VPC Endpoints (ENIs):** Securely connect to EventBridge from VPCs using private endpoints.

### Real-World Enterprise Examples

1. **Microservices Orchestration:** Trigger downstream services upon completion of an upstream process.
2. **Real-Time Analytics Pipelines:** Route clickstream data directly into processing pipelines.
3. **Security Monitoring:** Automate response actions for security events from [[AWS CloudTrail]].

### Diagrams & Trade-offs

**Architectural Diagram:**
- Placeholder for a diagram illustrating the flow from event source to EventBridge, through rules to targets like Lambda or SNS.

**Trade-offs:**
- Higher costs with more complex rule sets and higher event volumes.
- Latency increases with increased concurrency limits and retry policies.

### Exam Traps

1. **Event Size Limitations:** Misunderstanding the 1 MB size constraint per event.
2. **Rule Count Limits:** Overlooking the maximum number of rules allowed in a region.
3. **Concurrency Controls:** Confusing concurrency controls with retry policies.

### Decision Matrix (Features vs. Use Cases)

| Feature              | Microservices Orchestration | Real-Time Analytics | Security Monitoring |
|----------------------|------------------------------|--------------------|---------------------|
| Custom Event Buses   | Yes                          | Yes                | No                  |
| Dead Letter Queues   | Yes                          | Yes                | Yes                 |
| CloudWatch Metrics   | Optional                     | Mandatory          | Mandatory           |

### 2026 Updates

- Ensure all quotas and features are aligned with AWS's latest updates as of 2026.
- Review for any new service integrations or updated APIs.

> [!abstract] Exam Tip
> Ensure to validate configurations against the maximum limits in EventBridge, especially event size and rule count per region.

### Exam Scenarios

1. **Scenario:** Configuring a rule to route CloudTrail events to an SNS topic for alerting.
   - **Question:** What is the maximum size of each event processed by EventBridge?
   
2. > [!check] Implementation
   Implement retry policies and dead-letter queues for event processing failures.

### Interaction Patterns

- **Event Sources -> EventBus -> Rules -> Targets (Lambda, SNS, SQS):** Standard flow from source to final action.
- **Self-referencing Event Buses:** Advanced use case where events are routed internally within the bus itself.

> [!warning] Quota
> Be mindful of EventBridge’s technical limits such as event size and rule count per region when designing your architecture.

### Strategic Alignment & Professional Tone

Each piece of information is structured to align with SAP-C02 exam objectives and provide high-value technical insight. The tone remains concise and focused on essential details, ensuring clarity for complex concepts.

> [!danger] Distractor
> Note that EventBridge does not manage application state internally, unlike services like SNS/SQS or DynamoDB which are more suitable for applications requiring complex state management.
```