## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[lambda|AWS Lambda]] is a serverless compute service that lets you run your code without provisioning or managing servers. [[lambda]] automatically scales your applications in response to incoming event traffic. The key [[RDS_Instance_Types|internals]] include:

- **Function Execution Environment**: A container image with runtime and your function code. [[lambda]] reuses these environments to serve multiple requests, reducing cold start time.
- **Container Image Replication**: To minimize latency during scale-up, [[lambda]] proactively distributes container images across multiple Availability Zones (AZs) within a region.
- **Concurrency Management**: [[lambda]] manages concurrent executions by allocating resources from a shared pool of underlying hardware. This abstraction simplifies horizontal scaling and resource allocation.

To achieve **global scale**, you can deploy [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] in multiple regions and leverage AWS's **Global Accelerator**. Global Accelerator provides a single entry point for users worldwide, routing their requests to the nearest geographically distributed regional endpoint for lower latencies.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service           | Suitable Use Cases                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------- |
| [[lambda]]           | Real-time file processing, web apps, [[iot]] device telemetry analysis, mobile backend services            |
| [[Fargate]]          | Containerized microservices requiring fine-grained control over compute resources                |
| Batch            | Data processing tasks with high memory requirements, ad hoc workloads, ML model training             |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]]       | Containerized web applications requiring minimal management                                              |
| ECS/EC2          | Applications needing more direct control over infrastructure and OS                                   |
|