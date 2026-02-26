```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[Amazon Kinesis Data Streams]] Capacity Planning

#### UNDER-THE-HOOD MECHANICS:
[[Amazon Kinesis Data Streams]] is designed to handle high throughput and large volumes of streaming data. It operates by dividing the stream into shards, where each shard can process up to 1 MB per second of data or 1000 records per second.

- **Data Flow**: Data producers send data as records to a stream. Each record is distributed among one of the shards in the stream based on partition keys.
- **Consistency Models**: [[Amazon Kinesis Data Streams]] provides at-least-once delivery and does not guarantee strict ordering within the same shard, but ordering can be maintained by ensuring that all records from a single source use the same sequence number.
- **Underlying Infrastructure Logic**: Data is replicated across multiple availability zones for fault tolerance. Each shard has a leader node that handles writes and replicas that handle reads.

#### EXHAUSTIVE FEATURE LIST:
1. **Shards**: Units of throughput capacity, with each shard providing 1 MB/second data input or 1000 records per second.
2. **Partition Keys**: Used to determine which shard a record will be sent to.
3. **Sequence Numbers**: Unique identifiers for each record within a shard.
4. **Data Retention Periods**: Can be set from 24 hours up to 8760 hours (1 year).
5. **Enhanced Fan-Out**: Allows a single shard to support multiple consumers with low-latency reads.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Define Data Requirements**: Determine the expected data volume and velocity.
2. **Estimate Shards**: Calculate required shards based on the throughput requirements (1 MB/second or 1000 records per second).
3. **Create Stream**: Use [[AWS Management Console]], SDKs, or CLI to create a stream with the appropriate number of shards.
4. **Provisioning**: Adjust shard count based on workload changes using `UpdateShardCount` API.
5. **Data Ingestion**: Producers push data into the stream using partition keys.
6. **Data Processing**: Consumers read from the stream and process records.

#### TECHNICAL LIMITS (2026):
- Maximum of 10,000 shards per account.
- Each shard can handle up to 1 MB/second or 1000 records per second.
- Maximum data retention period is 8760 hours (365 days).

#### CLI & API GROUNDING:
```sh
# Create Stream
aws [[kinesis]] create-stream --stream-name <name> --shard-count <count>

# Update Shard Count
aws [[kinesis]] update-shard-count --stream-name <name> --target-shard-count <new_count> --scaling-type UNIFORM_SCALING

# Describe Stream
aws [[kinesis]] describe-stream --stream-name <name>
```

#### PRICING & TCO:
- Cost is based on the number of shards used and data processed.
- Optimization Strategies: Right-size shard count, use data compression (zlib or GZIP), and leverage [[AWS Pricing Calculator]] to estimate costs.

#### COMPETITOR COMPARISON:
- **Apache Kafka**: More flexible but requires more operational overhead for scaling and management.
- **Google Pub/Sub**: Event-based messaging service with higher latency compared to Kinesis.

**Architectural Trade-offs**:
- [[Amazon Kinesis Data Streams]] is easier to manage at scale but has fixed throughput per shard, which can be limiting in highly variable workloads.

#### INTEGRATION:
- **[[cloudwatch]]**: Metrics for stream performance and health.
- **IAM**: Role-based access control for producers and consumers.
- **VPC**: Can set up endpoints within a VPC for secure network isolation.

#### USE CASES:
1. **Real-time Analytics**: Processing clickstream data in real time.
2. **IoT Data Ingestion**: Handling sensor data from IoT devices.

#### EXAM TRAPS:
1. Misunderstanding the difference between partition keys and sequence numbers.
2. Overestimating or underestimating required shard count based on throughput needs.

#### DECISION MATRIX:
| Feature              | Use Case Example  |
|----------------------|--------------------|
| Shards               | Real-time analytics for high throughput data. |
| Partition Keys       | Ensuring correct distribution of records among shards. |

#### EXAM SCENARIOS:
1. Determining the right number of shards based on given throughput requirements.
2. Configuring data retention periods for compliance needs.

> [!abstract] Exam Tip
> Ensure you understand the difference between partition keys and sequence numbers to avoid common exam pitfalls.

> [!check] Implementation
> Use `UpdateShardCount` API to adjust shard count dynamically according to workload changes.

> [!danger] Distractor
> Be aware of potential misuses such as using Kinesis for batch processing or infrequent updates with low throughput needs, which can complicate your solution unnecessarily.

> [!warning] Quota
> Keep in mind the technical limits: maximum of 10,000 shards per account and each shard handles up to 1 MB/second or 1000 records per second.
```