**Advanced Architecture**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] is a content delivery network (CDN) that speeds up distribution of static and dynamic web content, such as .html, .css, .js, and image files, to end users. It delivers content through a worldwide network of edge locations, which enables low latency and high data transfer rates.

[[RDS_Instance_Types|Internals]]: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] uses a globally distributed network of edge servers to cache and deliver content to end users. When a user requests content from a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution, the request is routed to the nearest edge server. If the requested content is already cached at the edge server, it is delivered directly to the user. Otherwise, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] retrieves the content from the origin server and caches it at the edge server before delivering it to the user.

[[RDS_Instance_Types|Global Scale Considerations]]: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] automatically scales to meet the demands of a global audience. It can handle large amounts of traffic and distribute content to millions of users simultaneously. To achieve this level of scalability, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] uses a highly available and redundant architecture with multiple layers of [[api-gateway|caching]].

Under the Hood Mechanics: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] uses several techniques to optimize performance and reduce latency. These include:

* Content prefetching: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] prefetches content that is likely to be accessed in the near future, reducing the time it takes to load the content.
* TCP/IP optimization: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] uses advanced networking technologies, such as IPv6 and HTTP/2, to improve the performance of web applications.
* Cache invalidation: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] allows you to invalidate cached content, ensuring that users always see the most up-to-date version of your content.

**Comparison & Anti-Patterns**

| Service | Use Cases |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] | Global distribution of static and dynamic web content |
| [[ALB]] | Load balancing and SSL termination |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] | Domain name registration and DNS resolution |

Anti-patterns:

* Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to serve private or sensitive content without proper [[appsync|security]] measures in place.
* Configuring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to serve content from a single origin server without considering failover or backup options.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudfront:CreateInvalidation"
            ],
            "Resource": [
                "arn:aws:cloudfront::*:distribution/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudfront:Get*"
            ],
            "Resource": [
                "arn:aws:cloudfront::*:distribution/*",
                "arn:aws:cloudfront::*:invalidation/*"
            ]
        }
    ]
}
```
Cross-Account Access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:root"
            },
            "Action": [
                "cloudfront:CreateDistribution",
                "cloudfront:UpdateDistribution"
            ],
            "Resource": "arn:aws:cloudfront::111122223333:distribution/*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpce": "vpce-12345678"
                }
            }
        }
    ]
}
```
Organization SCPs:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "cloudfront:CreateDistribution",
                "cloudfront:DeleteDistribution",
                "cloudfront:UpdateDistribution"
            ],
            "Resource": [
                "arn:aws:cloudfront::*:distribution/*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalArn": [
                        "arn:aws:iam::111122223333:role/cloudfront-admin"
                    ]
                }
            }
        }
    ]
}
```

**Performance & Reliability**

Throttling Limits:

* Distribution Creation: 50 distributions per account per 24 hours
* Invalidation: 1000 paths per distribution per 24 hours

Exponential Backoff Strategies:
```yaml
1. Start with a base delay of 1 second.
2. Double the delay after each failed attempt until reaching the maximum delay of 60 seconds.
3. Wait for the maximum delay period before retrying.
```
HA/DR Patterns:

* Active-Active: Configure multiple origins and use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] geographic routing to route users to the nearest origin.
* Active-Passive: Configure a primary origin and a secondary origin for failover.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:

* Specify the duration of objects in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] cache (default: 24 hours)
* Enable price class options to minimize costs
* Use alternate domain names instead of default \*.[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cloudfront]].net domain names

Calculation Examples:

* Assume a distribution with 1 TB of data transferred per month, and a price class of "US, Canada, Europe". The monthly cost would be $0.15 per GB x 1024 GB = $153.60.
* Assume the same distribution with an additional 1000 invalidations per month. The monthly cost would be $0.001 per invalidation x 1000 invalidations = $1.00.

**Professional Exam Scenarios**

Scenario 1: A company has a website that serves static content from an Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. They want to improve the performance and reduce the latency of their website by using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. However, they also want to ensure that only authorized users can access the content. How should they configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to meet these requirements?

Correct Answer: Create a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution with the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket as the origin, enable restricted access with signed URLs, and restrict viewer access to specific IP addresses.

Incorrect Answer: Configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to serve the content directly from the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket without any restrictions.

Justification: The correct answer ensures that only authorized users can access the content while improving performance and reducing latency. The incorrect answer does not provide any [[appsync|security]] measures to prevent unauthorized access to the content.

Scenario 2: A company has a high-traffic website that serves dynamic content generated by an application running on [[ec2]] instances. They want to use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to distribute the content globally and reduce the load on their [[ec2]] instances. However, they are concerned about the potential impact of throttling limits. How should they mitigate these concerns?

Correct Answer: Implement an exponential backoff strategy when creating or updating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions.

Incorrect Answer: Increase the number of [[ec2]] instances to handle the increased traffic.

Justification: The correct answer provides a way to mitigate the impact of throttling limits by implementing an exponential backoff strategy. The incorrect answer does not address the throttling limits and may result in higher costs due to increased [[ec2]] instance usage.