### Advanced Architecture
At its core, [[appsync]] is a serverless GraphQL API that uses [[lambda|AWS Lambda]], Amazon [[dynamodb]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] to provide a flexible, scalable, and secure method of building scalable APIs. It utilizes the GraphQL protocol, which allows clients to specify their queries using a schema, resulting in efficient data transfers and reduced latency.

Internally, [[appsync]] supports several features such as real-time subscriptions via WebSockets, offline functionality through [[appsync|AWS AppSync]] client libraries, and automatic schema generation from REST endpoints. Additionally, it can be configured to perform request/response transformations using Velocity templates.

When considering global scale, [[appsync]] can be deployed across multiple regions by setting up separate API instances in each region. However, there are no built-in mechanisms for automatic data replication or failover between these instances. To achieve active-active deployments, you must implement custom solutions using AWS services like [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]], [[glue|AWS Glue]], or [[dms]].

### Comparison & Anti-Patterns
Here's a comparison table outlining when not to use [[appsync]] and potential alternatives:

| Scenario | Not Recommended | Alternative |
|---|---|---|
| Non-GraphQL APIs | [[appsync]] | RESTful APIs (e.g., [[api-gateway|API Gateway]]) |
| Strict JSON Schema validation | [[appsync]] | AWS Service Cataldyt |
| High-throughput event processing | [[appsync]] | AWS EventBridge/Kinesis |
| Real-time messaging without GraphQL | [[appsync]] | [[iot|AWS IoT]] Core/EventBridge |

Common anti-patterns include:

* Using [[appsync]] solely for CRUD operations without leveraging its GraphQL capabilities.
* Implementing custom authentication/authorization logic instead of utilizing AWS [[cognito]] or an IAM-based authorizer.

### [[appsync|Security]] & Governance
[[appsync]] provides fine-grained [[appsync|security]] configurations using AWS Identity and Access Management ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]) roles, AWS [[cognito]] user pools, and GraphQL-specific authorization rules.

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve permissions for [[appsync]] to interact with other AWS resources, such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], [[dynamodb|DynamoDB tables]], and [[sns]] topics. Here's an example policy granting [[appsync]] permission to invoke a specific [[lambda]] function:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME"
        }
    ]
}
```

Cross-account access can be implemented using AWS [[sts]] AssumeRole API calls within your [[appsync]] resolvers. For stricter governance, you can utilize [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to limit [[appsync]] usage at the organization level.

### Performance & Reliability
[[appsync]] has throttling limits based on the number of concurrent requests, which can be increased using AWS Support. In case of exceeding these limits, implementing exponential backoff strategies can help mitigate the impact of throttled requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], [[appsync]] should be deployed across multiple regions with dedicated resources in each region. By doing so, you can minimize the risk of single-region failures affecting your entire system.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls in [[appsync]] can be achieved by monitoring and adjusting settings like API [[api-gateway|caching]], data source connections, and throttling limits. Calculating costs involves understanding the pricing components:

* **API Calls**: Billed per call, depending on the operation type (query, mutation, or subscription).
* **Real-Time Subscriptions**: Additional charges apply for long-lived connections due to WebSocket usage.
* **Data Source Connections**: Charges depend on the underlying connected service ([[dynamodb]], [[lambda]], etc.).

### Professional Exam Scenarios
**Scenario 1:** A company wants to build a highly available [[appsync]] API with global read scalability while minimizing costs. Which architecture would you recommend?

![AppSync Multi-Region](https://mermaid.ink/img/eyJjb2RlIjoic2VuZHNvcnQgdmFyaWFibGVzLDEwMTIyNTMsIDQzOjEsIDMyOjE3LCAxOjUwLCAxOjkwLCAyOjU2LCAyOjU4XSwgInVybCI6Imh0dHA6Ly9nb2tzU3VwcG9ydC1yZWdzL3NvdXJjZS5qcGcifQ.png)

Correct answer: The recommended architecture separates read and write operations using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to distribute traffic across multiple [[appsync]] API instances. This setup ensures high availability while reducing costs compared to running all instances in a single region.

**Scenario 2:** An organization needs to enforce strict cost management [[policies]] for [[appsync]] usage across its AWS accounts. Which approach would best meet these requirements?

Correct answer: Utilize [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to restrict [[appsync]] usage at the organization level. Configure SCPs to allow only necessary actions related to [[appsync]], such as creating APIs and connecting to approved data sources.