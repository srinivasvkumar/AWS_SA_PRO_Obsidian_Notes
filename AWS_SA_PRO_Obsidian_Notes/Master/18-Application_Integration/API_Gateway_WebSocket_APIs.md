### Advanced Architecture
At its core, [[api-gateway|API Gateway]] WebSocket APIs provide a scalable and reliable way to manage WebSocket connections between clients and servers. The architecture consists of two main components: the connection manager and the message router.

The connection manager handles creating and managing individual WebSocket connections, distributing incoming requests to the appropriate backend server, and handling traffic during failovers or high availability scenarios. It also provides features like connection pinging and timeout management.

The message router enables routing messages between connected clients and backend services. This component is responsible for ensuring that messages are delivered to their intended recipients even when they switch sub-protocols or move across different availability zones within a region.

When dealing with global applications, you can distribute your WebSocket APIs using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] to register and discover services across multiple accounts and regions. By doing so, you create a consistent experience for users worldwide while minimizing latency.

### Comparison & Anti-Patterns

| Service               | Use Case                                           |
| --------------------- | ------------------------------------------------- |
| [[api-gateway|API Gateway]] REST API  | Stateless HTTP(S) communication, synchronous calls |
| [[lambda]]                | Event-driven, serverless compute                   |
| [[sqs]]                   | Message queues, asynchronous processing             |
| [[kinesis|Kinesis Data Streams]]  | Real-time data streaming                          |

Anti-patterns include using WebSocket APIs when stateless communication would suffice or trying to implement complex pub/sub systems without additional tools such as [[mq|Amazon MQ]] or [[sns]].

### [[appsync|Security]] & Governance

To secure an [[api-gateway|API Gateway]] WebSocket API, you may apply complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] at various levels. Here's an example granting `Connect` and `Disconnect` permissions to a specific user:

```json
{
  "Effect": "Allow",
  "Action": ["execute-api:ManageConnections"],
  "Resource": "arn:aws:execute-api:us-east-1:123456789012:*/*/@connections/*",
  "Condition": {
    "StringEqualsIfExists": {
      "aws:SourceVpc": "vpc-12345678"
    },
    "ArnLikeIfExists": {
      "aws:SourceArn": [
        "arn:aws:iam::123456789012:role/service-role/MyWebSocketRole",
        "arn:aws:sns:us-east-1:123456789012:MyTopic"
      ]
    }
  }
}
```

Cross-account access can be achieved by adding proper resource [[policies]] in the destination account and configuring the source account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles accordingly. Additionally, Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] can enforce restrictions on API usage across member accounts.

### Performance & Reliability

Each [[api-gateway|API Gateway]] WebSocket API has throttling limits enforced through tokens. To avoid exceeding these limits, implement exponential backoff strategies in case of failed requests. For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes, ensure each environment spans at least two availability zones.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented by enabling [[api-gateway|API Gateway]] usage plans, which allow charging based on the number of API calls and data transferred. Calculate costs using the [AWS Pricing Calculator](https://calculator.aws/) and monitor usage through AWS [[billing|Cost Explorer]].

### Professional Exam Scenario: Multi-Account Strategy

Imagine a company with separate development, staging, and production environments distributed across three accounts. They want to build a real-time chat application using [[api-gateway|API Gateway]] WebSocket APIs. How should they structure their resources?

1. Create a central organization account containing Service Control [[policies]] (SCPs) restricting API usage.
2. Set up cross-account access from the development, staging, and production accounts to the central organization account.
3. Register WebSocket API resources in the central organization account using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]].
4. Implement fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] controlling access to the individual WebSocket APIs in each account.

##### Correct Answer Justification

This approach ensures centralized control over API usage while maintaining logical separation between development, staging, and production environments. Cross-account access allows sharing of common resources while adhering to [[appsync|security]] [[iam|best practices]].

##### Incorrect Answers & Justifications

* Mixing all resources in one account: Poor separation of concerns and increased risk due to lack of granular control and visibility.
* Creating duplicate resources in each account: Wasteful and error-prone, increasing operational complexity and maintenance efforts.