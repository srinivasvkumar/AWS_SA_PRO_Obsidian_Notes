**Advanced Architecture**

Migration Hub is a central location to track and manage your application migrations across multiple AWS accounts and tools. It supports migration tools like Application Discovery Service, Server Migration Service, and Database Migration Service.

Internally, Migration Hub uses [[organizations|AWS Organizations]] to create a hierarchy of AWS accounts. This allows for consolidated management, visibility, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]. Global scale is achieved using AWS Region-specific services that communicate with the Migration Hub through API calls and events. The data flow between Migration Hub and integrated tools follows these steps:

1. Integration: Tools send migration status updates to Migration Hub via APIs or SDKs.
2. Aggregation: Migration Hub collects, normalizes, and stores metadata from integrated tools.
3. Visualization: Users can view migration progress in real-time using the Migration Hub console.

For advanced architectures, consider implementing these features:

- **Multi-account strategies**: Connect multiple AWS accounts within an [[AWS Organization]] to monitor migrated resources centrally.
- **Quotas**: Monitor and control usage of migration tools by setting individual quotas per account.
- **Integration**: Implement event-driven architectures using Amazon [[eventbridge]], allowing integration with custom applications and workflows.

**Comparison & Anti-Patterns**

| Factor                 | Migration Hub          | Alternatives                | Anti-Patterns             |
| ----------------------- | ---------------------- | ---------------------------- | -------------------------- |
| Tracking multiple tools | Centralized tracking   | Custom solution              | Mixing dev/prod resources   |
| Scalability            | Highly scalable        | Depends on custom solution    | Limited monitoring scope    |
| Cost                   | Free                   | Potentially higher costs     | Overlooking cost optimizations |

Common anti-patterns include mixing development and production resources within a single Migration Hub, making it difficult to maintain proper resource separation and [[appsync|security]] [[policies]].

**[[appsync|Security]] & Governance**

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], you can use JSON snippets like this one granting cross-account access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111111111111:root"},
      "Action": ["mgh:*"],
      "Resource": "*",
      "Condition": {"StringEquals": {"aws:SourceVpce": "vpce-1234567890abcdef0"}}
    }
  ]
}
```
Cross-account access can be further restricted using Organization Service Control [[policies]] (SCPs) to limit which actions are allowed within the Migration Hub.

**Performance & Reliability**

Migration Hub has throttling limits based on the number of requests per second. To ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your Migration Hub instances across different regions, and handle throttling exceptions using exponential backoff strategies.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented by managing migration tool quotas and monitoring associated costs. For example, set up [[billing]] alarms for specific thresholds, and [[nonportable_instructions|review]] monthly bills to identify unusual activity.

**Professional Exam Scenarios**

Scenario 1:
A company wants to track migrations across multiple AWS accounts while ensuring proper resource separation and [[appsync|security]]. They also want to minimize management overhead and integrate their existing monitoring system.

Correct answer: Utilize Migration Hub with [[organizations|AWS Organizations]] and connect custom monitoring systems using Amazon [[eventbridge]].

Incorrect answer: Create a custom solution without leveraging Migration Hub, resulting in increased management overhead and reduced feature compatibility.

Scenario 2:
An organization needs to track its database migrations but requires separate access permissions for each team member.

Correct answer: Set up individual AWS accounts for each team member within an [[AWS Organization]], and then connect them to the Migration Hub. Define granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and apply SCPs to enforce least privilege principles.

Incorrect answer: Share a single AWS account among all team members, increasing the risk of unauthorized access and violating [[iam|best practices]] for [[appsync|security]] and governance.