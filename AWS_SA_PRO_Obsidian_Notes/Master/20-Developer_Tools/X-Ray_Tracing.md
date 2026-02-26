### Advanced Architecture

At its core, X-Ray traces distributed applications by collecting data from various services and providing an interactive map of request flows. The primary components include:

- **Daemons**: [[xray|AWS X-Ray]] daemon (`awsxray-daemon`) receives and processes tracing data from services like [[lambda]], [[ec2]] instances, and containers. It can also be self-hosted for on-premises workloads.
- **Service Proxy**: A reverse proxy that intercepts incoming requests and outgoing responses, injecting sampling and instrumentation directives into application code without modifying it.
- **Sampling Rules**: Define which subset of requests should be traced based on attributes such as HTTP headers, URL paths, or cookies.
- **Grouping**: Organize traces using tags, dimensions, and metadata for better visualization and filtering.

For global scale applications, you may deploy multiple X-Ray daemons across different regions or even accounts. To minimize latency and egress costs, consider placing daemons near the source of tracing data. For example, if your application runs on [[ecs]] [[Fargate]] in us-west-2, install the X-Ray daemon within the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

### Comparison & Anti-Patterns

| Service              | Use Case                                                                   |
| -------------------- | ------------------------------------------------------------------------- |
| **[[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]**       | Audit management & compliance reporting                                    |
| **[[cloudwatch]]**       | Log monitoring, alerting, and performance metrics                          |
| **Application Insights** | Real-time observability for .NET applications                            |
| **ElasticSearch**    | Centralized [[vpc-flow-logs|logging]], search, and analysis                                |
| **Distributed Tracing** | Analyzing latencies, [[api-gateway|errors]], and performance issues in distributed systems |

Anti-pattern: Using X-Ray for long-term storage or archival purposes. Instead, leverage [[cloudwatch|CloudWatch Logs]] or ElasticSearch for log aggregation and retention requirements.

### [[appsync|Security]] & Governance

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "xray:BatchGetTraces",
        "xray:GetTraceSummaries",
        "xray:PutTelemetryRecords",
        "xray:PutTraceSegments"
      ],
      "Resource": "*"
    }
  ]
}
```
Cross-Account Access:

To enable cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to submit trace segments and telemetry records. Then, attach a trust policy allowing the destination account's X-Ray daemon to assume the role.

Organization SCPs:

Implement Service Control [[policies]] (SCPs) at the organization level to enforce [[control-tower|guardrails]] around X-Ray usage. For instance, restrict specific actions, resources, or API operations based on organizational units (OUs) or individual accounts.

### Performance & Reliability

Throttling Limits:

X-Ray imposes throttling limits on various APIs, such as `BatchGetTraces`, `GetTraceSummaries`, and `PutTelemetryRecords`. To avoid hitting these limits, implement exponential backoff strategies when making repeated calls.

HA/DR Patterns:

For high availability, distribute X-Ray daemons across multiple availability zones within a region. For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], ensure that daemons are deployed in standby mode in secondary regions. Upon failure detection, promote the standby daemons to active state.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls:

Use X-Ray's [[cost-allocation-tags|cost allocation tags]] to track spending by project, team, or application. This allows you to identify potential cost savings opportunities and optimize resource utilization.

Calculation Examples:

Assume an organization uses X-Ray in two projects: ProjectA and ProjectB. They allocate 60% and 40% of total X-Ray costs to each project respectively.

ProjectA's monthly X-Ray cost = Total X-Ray cost \* 0.6
ProjectB's monthly X-Ray cost = Total X-Ray cost \* 0.4

### Professional Exam Scenarios

Scenario 1:
A company wants to analyze latency issues in their serverless application running on [[lambda|AWS Lambda]]. They have multiple functions interacting with [[dynamodb|DynamoDB tables]] and [[api-gateway|API Gateway]] endpoints. What solution would you propose?

Correct Answer: Implement [[xray|AWS X-Ray]] tracing for the serverless application. By default, [[lambda]] integrates with X-Ray, so no additional configuration is needed. However, you must configure X-Ray to support other services involved in the application flow, such as [[api-gateway|API Gateway]] and [[dynamodb]].

Incorrect Answer: Configure AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] to monitor API calls and capture logs. While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] provides valuable audit and compliance insights, it does not offer real-time observability into application performance issues.

Scenario 2:
An organization has implemented X-Ray for their production environment but noticed increased egress costs due to data transfer between regions. How can they optimize costs while maintaining X-Ray functionality?

Correct Answer: Deploy X-Ray daemons closer to the source of tracing data. For example, if the application runs on [[ecs]] [[Fargate]] in us-west-2, install the X-Ray daemon within the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. This reduces egress costs by minimizing data transfers across regions.

Incorrect Answer: Store X-Ray traces locally using Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets. Although storing traces in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets can reduce egress costs, it does not address the underlying issue of data transfer between regions.