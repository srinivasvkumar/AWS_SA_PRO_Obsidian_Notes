**Advanced [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Architecture**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds within a developer-friendly environment. It works seamlessly with services like [[shield|AWS Shield]], [[Lambda@Edge]], Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] to provide powerful capabilities such as real-time protection against attacks, content transformation, and customized logic near the edge.

[[RDS_Instance_Types|Internals]]:
- **Distribution**: Represents a unique URL that acts as a front end to an origin server or a set of origin servers. Contains one or more cache behaviors which determine how [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] handles requests for specific URL paths.
- **Origin Server**: Can be any HTTP server that stores the definitive version of your objects. This could be an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket, Elastic Load Balancer, or a custom origin.
- **Cache Behavior**: Defines how [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] serves particular parts of your website or application. A distribution can have multiple cache behaviors, each mapping to different URL path patterns.
- **Edge Location**: Where content is cached and distributed to end users. There are two types: origin edge locations (where you create or update your distributions) and regional edge caches (which improve performance by storing copies of frequently accessed content closer to your viewers).

[[RDS_Instance_Types|Global Scale Considerations]]:
- **Geo-Restrictions**: Implemented via geographic restriction type in the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution settings. This restricts access to only those countries specified.
- **Path-based Configuration**: Allows fine-grained control over cache behavior based on URL paths. For example, directing traffic from /images/* to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket while sending all other requests to an [[elb]].

Under The Hood Mechanics:
- **Viewer Request Handling**: Viewer sends request to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]; if content isn't cached at the Edge location, it forwards the request to the Origin Server.
- **Origin Response [[api-gateway|Caching]]**: Upon receiving response from the origin server, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] caches the response at the Edge location before forwarding it to the viewer.

**Comparison & Anti-Patterns**

| Service | Use Case |
|---|---|
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] | Global distribution of static and dynamic web content, offload high volume traffic away from origin servers. |
| [[ALB]] + [[NLB]] | Internal load balancing, not meant for public internet exposure. |
| Direct Connect + [[Srinivas_Notes/VPC|VPC]] Endpoint | Private connectivity between [[Srinivas_Notes/VPC|VPC]] resources without traversing the public internet. |
| [[Srinivas_Notes/S3|S3]] Transfer Acceleration | Point-to-point transfers directly into or out of [[Srinivas_Notes/S3|S3]] buckets, typically for large files or infrequent uploads. |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] purely for SSL termination when cheaper options exist (like [[ALB]] or [[NLB]]), or assuming all content will automatically be faster due to CDN [[api-gateway|caching]] without considering underlying architecture.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Effect": "Allow",
    "Action": ["cloudfront:CreateInvalidation"],
    "Resource": "arn:aws:cloudfront::*:distribution/*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVpce": "vpce-1234abcd"
        }
    }
}
```
Cross-Account Access:
- Trusted Signers: Allow named profiles or AWS accounts to create invalidations on your behalf.
- Origin Access Identity: Grant read permissions to your [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket so it can be used as a valid origin.

Organization SCPs:
- Prevent creation of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions outside of specific OUs or accounts.

**Performance & Reliability**

Throttling Limits:
- Maximum number of distributions per account: 200.
- Requests per second per single edge location: 10,000.

Exponential Backoff Strategies:
- Handle potential throttling by implementing exponential backoff during API calls.

HA/DR Patterns:
- Use multiple origins to ensure high availability. In case of failure, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] automatically routes traffic to healthy origins.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:
- Enable price class controls to limit serving content only from certain regions.
- Leverage [[cloudfront|Field-Level Encryption]] to reduce egress costs when transmitting sensitive fields over HTTPS.

Calculation Examples:
- Assume 10TB data transfer monthly, US East (Ohio) region, 10% dynamic content.
  - Static Data Transfer: 9TB * $0.085 = $765
  - Dynamic Data Transfer: 1TB * $0.150 = $150
  Total Monthly Cost: $915

**Professional Exam Scenarios**

Scenario 1:
Your company needs to serve static assets (images, CSS, JavaScript) for a global user base but wants to minimize latency while keeping costs under control. They already host these assets on an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. What solution would you propose?

Correct Answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] with an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket as origin configured to serve only the /static/* URL path pattern. This reduces latency by [[api-gateway|caching]] frequently accessed content closer to end users and minimizes data transfer costs.

Incorrect Answers:
a) Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration: Not suitable for regular web content delivery.
b) Serve directly from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]: Increases latency due to lack of [[api-gateway|caching]] at the edge.

Scenario 2:
To meet regulatory requirements, your organization must prevent users from certain countries from accessing corporate intranet sites served through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. How would you implement this?

Correct Answer: Utilize Geo-Restrictions feature available in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] settings to block specified countries from accessing the site.

Incorrect Answers:
a) Implement IP whitelisting: Insufficient since IP addresses can change or be shared among multiple countries.
b) Move intranet sites behind Direct Connect: Overkill for simple geographic restrictions.