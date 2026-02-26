## [[RDS_Instance_Types|1. Advanced Architecture]]
[[sns]] (Simple Notification Service) is a fully managed messaging service that allows for the sending of messages or calls to distributed systems, services, and clients. It supports multiple message brokers, including HTTP/S, Email, SMS, Mobile Push, and AMQP. Understanding its [[RDS_Instance_Types|internals]] is crucial to building scalable and performant solutions.

### [[RDS_Instance_Types|Global Scale Considerations]]
[[sns]] operates at a global scale by replicating topics across regions. This ensures low-latency delivery when subscribers are spread across different geographical locations. To implement a global fanout pattern, you can publish to a single "topic" in one region, which then automatically replicates the message to corresponding topics in other regions. The following Mermaid diagram illustrates this architecture:
```mermaid
graph LR
    Subscriber[Subscriber]
    Topic[Topic]