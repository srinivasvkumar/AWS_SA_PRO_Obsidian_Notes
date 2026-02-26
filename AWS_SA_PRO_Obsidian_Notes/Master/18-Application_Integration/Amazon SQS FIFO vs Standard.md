```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Amazon SQS FIFO vs Standard

#### UNDER-THE-HOOD MECHANICS:

- **[[Internal Data Flow]]**:
  - **[[Standard Queue]]**: Uses at-least-once delivery model, where messages can be delivered multiple times if the visibility timeout expires before processing.
  - **[[FIFO Queue]]**: Guarantees that messages are processed in the exact order they were sent. It uses sequence numbers to ensure strict ordering and exactly-once processing.

- **[[Consistency Models]]**:
  - **[[Standard Queue]]**: At-least-once delivery model ensures high throughput but can introduce message duplication.
  - **[[FIFO Queue]]**: Exactly-once processing with strong consistency by using content-based deduplication and message group IDs for ordered batches.

- **Underlying Infrastructure Logic**:
  - Both types of queues are managed services hosted on [[Amazon's infrastructure]]. FIFO queues have additional logic to enforce strict ordering, while standard queues prioritize high throughput over message order.

#### EXHAUSTIVE FEATURE LIST:

- **[[Standard Queue Features]]**:
  - At-least-once delivery
  - High throughput (up to millions of messages per second)
  - Visibility timeout and dead-letter queue support

- **[[FIFO Queue Features]]**:
  - Exactly-once processing guarantee
  - Content-based deduplication for message uniqueness
  - Message grouping by group IDs for ordered batches within a queue
  - FIFO ordering within the same group ID

#### STEP-BY-STEP IMPLEMENTATION:

1. **Create [[sqs]] Queues**: Use AWS Management Console, CLI, or SDK to create standard and FIFO queues.
   ```sh
   aws sqs create-queue --queue-name my-standard-queue
   aws sqs create-queue --queue-name my-fifo-queue.fifo --attributes file://fifo-attributes.json
   ```
2. **Configure Queue [[policies]]**: Set appropriate [[00_Master_Links_and_Intro|IAM]] [[policies]] for message sending and receiving permissions.
3. **Send Messages**:
  - Standard: Use `sendMessage` API without special attributes.
    ```sh
    aws sqs send-message --queue-url <standard_queue_url> --message-body "Message content"
    ```
  - FIFO: Include `MessageGroupId` and optionally `MessageDeduplicationId`.
    ```sh
    aws sqs send-message --queue-url <fifo_queue_url> --message-body "Message content" --message-group-id "group1" --message-deduplication-id "<unique_id>"
    ```
4. **Receive Messages**: Use `receiveMessage` API to process messages from both queue types.
5. **Delete Messages**: After processing, delete messages using their receipt handle.

#### TECHNICAL LIMITS (2026):

- **[[Standard Queue]]**:
  - Maximum number of messages: No limit
  - Message size: Up to 256 KB
  - Visibility timeout: Adjustable between 0 and 43200 seconds

- **[[FIFO Queue]]**:
  - Maximum number of messages: No limit
  - Message size: Up to 256 KB
  - Visibility timeout: Adjustable between 0 and 43200 seconds
  - Number of message groups per queue: Limited by [[service-quotas|AWS service quotas]]

#### CLI & API GROUNDING:

- **CLI Commands**:
    ```sh
    aws sqs list-queues
    aws sqs get-queue-url --queue-name <queue_name>
    aws sqs send-message --queue-url <queue_url> --message-body "<message>"
    aws sqs receive-message --queue-url <queue_url>
    aws sqs delete-message --queue-url <queue_url> --receipt-handle <handle>
    ```

- **API Flags**:
  - `sendMessage`: `--message-group-id`, `--message-deduplication-id`
  - `receiveMessage`: `MaxNumberOfMessages`, `VisibilityTimeout`

#### PRICING & TCO:

- **Cost Drivers**:
  - Message delivery, receive, and delete actions
  - Data transfer charges for incoming/outgoing traffic

- **Optimization Strategies**:
  - Use message batching to reduce API calls
  - Enable long polling to optimize costs by reducing unnecessary API requests
  - Configure proper queue [[policies]] to minimize unused resources

#### COMPETITOR COMPARISON:

- **[[Azure Service Bus]]**: Supports both FIFO and at-least-once delivery models with similar features but lacks some specific AWS [[api-gateway|integrations]].
- **[[Google Pub/Sub]]**: Provides ordering guarantees within a specific subscription, but not as granular as [[sqs]] FIFO.

#### INTEGRATION:

- **[[cloudwatch]]**: Use for monitoring queue metrics like message send/receive rates.
- **[[iam]]**: Define [[policies]] and roles to control access to queues.
- **[[VPC Endpoints]]**: Access queues from VPCs securely without exposing them publicly.

#### USE CASES:

1. **Real-time Order Processing**: FIFO queues ensure that order processing is handled in the correct sequence.
2. **Event [[vpc-flow-logs|Logging]]**: Standard queues handle high throughput with occasional message duplication being acceptable.

#### EXAM TRAPS:

- Misunderstanding visibility timeout impacts on message delivery in both queue types.
- Overlooking the need to set unique `MessageDeduplicationId` for strict deduplication in FIFO queues.

#### DECISION MATRIX:

| Features               | Use Case: Order Processing | Use Case: Event [[vpc-flow-logs|Logging]] |
|------------------------|----------------------------|-------------------------|
| Message Ordering       | Required                   | Not Required            |
| Throughput             | Moderate                   | High                    |
| Consistency Model      | Exactly-once               | At-least-once           |

#### 2026 UPDATES:

- Ensure all service limits, pricing updates, and CLI/API changes are aligned with AWS’s 2026 landscape.

#### EXAM SCENARIOS:

- Scenario: Given a use case that requires ordered message processing, select the appropriate [[sqs]] queue type.
- Scenario: Troubleshooting message duplication issues in standard queues vs. FIFO queues.

#### INTERACTION PATTERNS:

- Service-to-service interactions include API calls for sending/receiving messages and managing queue [[policies]].

#### STRATEGIC ALIGNMENT:

- Focus on high-weight exam topics like queue setup, message ordering guarantees, and integration with [[00_Master_Links_and_Intro|other AWS services]].

#### PROFESSIONAL TONE & AUDIENCE:

- Tailored specifically for SAP-C02 candidates with a concise, high-value delivery style.

#### PRIORITIZATION & CLARITY:

- Prioritize [[kms|key concepts]] and use cases relevant to the exam blueprint.
- Provide clear explanations and references to official AWS documentation.