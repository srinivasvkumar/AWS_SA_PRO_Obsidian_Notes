**Advanced Architecture**

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] is a globally distributed network of edge servers that cache and deliver content from the closest location to the end user. This is achieved through a process called "origin request handling" where [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] checks if the requested object exists in the nearest edge server before forwarding the request to the origin server. If it does, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] delivers the cached object directly from the edge server, reducing latency and improving performance.

When an origin request isn't found at the edge, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] sends the request to the origin server and caches the response at the edge. To secure these communications, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] uses HTTPS connections between viewers, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], and origin servers. It also supports Server Name Indication (SNI) which allows multiple custom SSL certificates to be associated with a single [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution.

For advanced users, [[Lambda@Edge]] enables writing JavaScript functions that run in response to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] events, allowing customizations such as modifying headers or URLs, IP address restriction, or A/B testing.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] | Content delivery with high availability, low latency, and [[appsync|security]] features like WAF, [[shield]], and Field Level Encryption. |
| [[ALB]] + [[route53]] | Application load balancing and DNS routing. Not ideal for static assets due to higher latency compared to [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. |
| [[Srinivas_Notes/S3|S3]] Transfer Acceleration | Point-to-point transfers to [[Srinivas_Notes/S3|S3]] from remote locations. Not suitable for general-purpose content delivery. |
| Global Accelerator | Improving performance for applications that have a complex origin infrastructure. Not recommended when [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] can serve the purpose. |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] only for SSL termination without taking advantage of [[transfer-family|other features]] like [[api-gateway|caching]] and edge computing. Another mistake is configuring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to distribute uncompressed content, increasing data transfer costs and affecting performance.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve managing permissions for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] actions, especially when dealing with multiple accounts. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "cloudfront:CreateDistribution",
              "cloudfront:GetDistribution",
              "cloudfront:UpdateDistribution",
              "cloudfront:DeleteDistribution"
            ],
            "Resource": "arn:aws:cloudfront::*:distribution/*"
        }
    ]
}
```

Cross-account access requires creating a trust relationship between two AWS accounts by adding the ARN of the trusted account to the principal field in the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy. Organizational Service Control [[policies]] (SCPs) can enforce restrictions across all member accounts within an organization.

**Performance & Reliability**

Throttling limits depend on the type of request. For example, GET requests have a limit of 15 million per month while POST requests have a lower limit of 3000 per month. In case of exceeded throttle limits, retrying after an exponential backoff strategy helps manage [[api-gateway|errors]] more efficiently.

HA/DR patterns involve deploying resources across different regions or availability zones. With [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], geo-routing can direct user traffic to specific edge servers based on their location, ensuring high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be applied using features like price classes, cache behaviors, and restricted bucket operations. Price classes determine which edge servers serve your content based on their proximity to the viewer, thus influencing cost. Cache behaviors let you configure how [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] handles requests for specific files, while restricting bucket operations prevents unnecessary data transfer costs.

Calculating costs involves understanding charges for data transfer, number of requests, and optional features used. Remember that there's no charge for first 10 TB of data transfer out to the internet and first 2 million requests per month.

**Professional Exam Scenarios**

Scenario 1: A company wants to serve large volumes of multimedia content via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] while maintaining strict control over who can access it. They already use Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storage and have a dedicated team for managing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]]. Propose a solution that meets these requirements.

Correct Answer: Implement a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution with Origin Access Identity (OAI) to restrict [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] access to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] only. Configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to require signed URLs or cookies for access. This ensures that only those with valid credentials can access the content.

Incorrect Answers: Using only OAI without signed URLs or cookies would not provide sufficient control over who can access the content. Similarly, relying solely on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]] for controlling access could expose the content publicly.

Scenario 2: A media organization streams live events globally using AWS Elemental MediaLive and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. The organization needs to minimize latency and optimize costs. Propose a solution architecture considering these factors.

Correct Answer: Set up a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution with real-time Edge Origin Latency optimization enabled. Serve the stream from a regional Edge Cache location close to the event venue. Enable Gzip compression to reduce data transfer costs. Finally, set up a price class that serves most users from the fewest number of locations feasible to minimize costs.

Incorrect Answers: Serving the stream from a single centralized origin server rather than using Edge Cache locations may result in increased latency. Disabling compression would increase data transfer costs unnecessarily.