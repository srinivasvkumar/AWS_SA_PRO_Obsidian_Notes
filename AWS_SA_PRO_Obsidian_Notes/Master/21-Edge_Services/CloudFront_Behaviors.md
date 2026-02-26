**[[RDS_Instance_Types|1. Advanced Architecture]]**

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] is a globally distributed network of edge servers that cache and deliver content from the closest location to the end user. The internal architecture consists of three main components:

a. **Origin Servers**: These can be any HTTP server (e.g., Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], Elastic Load Balancer, or custom origin) that stores the original version of your objects.
b. **Edge Locations**: Geographically dispersed data centers that store frequently requested content and serve as the first point of contact between users and your application.
c. **Regional Edge Caches (RECs)**: Intermediate [[api-gateway|caching]] layer between origins and edge locations, handling a significant portion of requests without needing to access the origin server.

When a request comes in, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] checks if it has a valid object in the REC. If not, it fetches the object from the origin server, caches it in the REC, and then serves it to the user from there. This process reduces latency and improves performance by serving content closer to the user.

[[RDS_Instance_Types|Global scale considerations]] include:

- *Distribution Types*: There are two types of distributions: Web and RTMP. Web distributions serve static and dynamic web content while RTMP is used for Adobe Flash media distribution.
- *Price Classes*: Customize costs by selecting specific edge locations based on their price class. This trade-offs cost savings against potential higher latencies.
- *Cache [[policies]]*: Define how long cached objects should stay in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] before checking for an updated version at the origin.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service             | Use Case                                                              |
| ------------------- | -------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]          | Content Delivery Network (CDN), low-latency, high-throughput        |
| Amazon [[Srinivas_Notes/S3|S3]] Transfer  | Large file transfers between [[Srinivas_Notes/S3|S3]] buckets, [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/privatelink|VPC endpoints]]                |
| Direct Connect     | Dedicated network connection between on-premises infrastructure and AWS |
| [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] | Improve performance for real-time applications like gaming, VoIP |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] only as a CDN without taking advantage of features such as [[Lambda@Edge]], Field Level Encryption, or Cache [[policies]].

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting permissions to various resources within [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. For example, allowing a user to create and manage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions but not modify existing ones:

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
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "aws:RequestedOperation": [
                        "CreateDistribution",
                        "GetDistribution",
                        "UpdateDistribution",
                        "DeleteDistribution"
                    ]
                }
            }
        },
        {
            "Effect": "Deny",
            "Action": [
                "cloudfront:CreateDistribution",
                "cloudfront:GetDistribution",
                "cloudfront:UpdateDistribution",
                "cloudfront:DeleteDistribution"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "NotStringEqualsIgnoreCase": {
                    "aws:RequestedOperation": [
                        "CreateDistribution",
                        "GetDistribution",
                        "UpdateDistribution",
                        "DeleteDistribution"
                    ]
                }
            }
        }
    ]
}
```

Cross-account access requires creating a trust relationship between the source account and destination account, followed by configuring appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions.

Organization Service Control [[policies]] (SCPs) limit actions performed by identity principals (users and roles) within member accounts. To restrict creation of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions outside of specific [[organizations|AWS Organizations]] OUs, use an [[SCP]] similar to:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "cloudfront:CreateDistribution",
                "cloudfront:CreateInvalidation",
                "cloudfront:Put*",
                "cloudfront:Get*",
                "cloudfront:Delete*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalOrgID": [
                        "<ORG_ID>"
                    ]
                }
            }
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the number of requests per second per edge location and the total number of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions. Handling throttling involves implementing exponential backoff strategies using SDKs or libraries that automatically retry API calls after waiting for increasing intervals.

HA/DR patterns require distributing traffic across multiple regions using geo-routing or active-active configurations with [[route53]] [[route53|health checks]].

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include setting up separate distributions for different environments (dev, staging, production) and leveraging price classes for optimizing costs. Calculating costs involves understanding factors such as data transfer out, number of requests, cache behavior settings, and field level encryption usage.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: A company wants to securely distribute sensitive data to their customers through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] while minimizing egress costs. How would you implement this?

Correct answer: Implement client-side [[cloudfront|field-level encryption]] using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] and [[Lambda@Edge]]. This encrypts specified fields within the response payload before sending them to the client. It also ensures minimal egress costs since decryption happens on the client side.

Incorrect answer: Using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] server-side encryption (SSE) would not minimize egress costs because encryption and decryption occur on the server side, requiring more bandwidth.

Scenario 2: An organization manages multiple AWS accounts and wants to enforce strict governance over [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] usage. What steps should they follow?

Correct answer: Create an [[AWS Organization]], set up Service Control [[policies]] (SCPs) to deny unwanted actions, and establish trust relationships between member accounts. Additionally, utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] to grant necessary access permissions.

Incorrect answer: Configuring individual [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] within each account without centralized management may lead to inconsistent governance practices and increased operational overhead.