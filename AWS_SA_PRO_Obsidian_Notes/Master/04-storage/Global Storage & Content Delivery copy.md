```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Global Storage & Content Delivery]]

#### UNDER-THE-HOOD MECHANICS:
[[Global storage]] and [[content delivery]] services like [[Amazon S3]], [[Amazon CloudFront]], and [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] operate on a distributed architecture designed to ensure low-latency access from anywhere globally.

- **[[Amazon S3]]**: 
  - Uses multiple availability zones (AZs) within regions for data replication.
  - Consistency models include eventual consistency and read-after-write consistency.
  
- **[[Amazon CloudFront]]**:
  - Distributes content through a global network of edge locations.
  - Caches origin content to reduce latency and load on the origin servers.

- **[[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]]**:
  - Utilizes a globally distributed network of endpoints (anycast IP addresses) for optimized routing.
  - Ensures low latency and high availability by directing traffic to the nearest optimal endpoint.

#### EXHAUSTIVE FEATURE LIST:

- **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**:
  - Versioning
  - Lifecycle [[policies]]
  - Cross-region replication
  - Intelligent-Tiering
  - Bucket [[policies]]

- **Amazon [[00_Master_Links_and_Intro|CloudFront]]**:
  - Real-time log streaming
  - [[Lambda@Edge]] for dynamic content manipulation
  - [[cloudfront|Field-level encryption]]
  - Geo-restriction

- **[[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]**:
  - Endpoints in multiple AWS regions
  - [[00_Master_Links_and_Intro|Application Load Balancer (ALB)]] and [[00_Master_Links_and_Intro|Network Load Balancer (NLB)]] support
  - DNS failover

#### STEP-BY-STEP IMPLEMENTATION:

1. **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Setup**: 
   - Create an [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] bucket.
   - Enable versioning, lifecycle [[policies]], and cross-region replication as needed.

2. **[[00_Master_Links_and_Intro|CloudFront]] Distribution**:
   - Create a [[cloudfront]] distribution with the [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] bucket as the origin.
   - Configure cache settings, [[cloudfront|field-level encryption]], and geo-restrictions.

3. **Global Accelerator Configuration**:
   - Register endpoints ([[ALB]] or [[NLB]]).
   - Create an accelerator and attach it to the endpoint group.

#### TECHNICAL LIMITS (2026):

- **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**: 
  - Maximum of 100,000 PUT/COPY/POST/DELETE requests per second per bucket.
  - Up to 2,048 lifecycle [[policies]] per account.

- **[[00_Master_Links_and_Intro|CloudFront]]**:
  - Max. number of distributions: 500 (can be increased with AWS support).
  - Max. number of cache behaviors: 100.

- **Global Accelerator**:
  - Up to 32 endpoints in an endpoint group.
  - One accelerator per region.

#### CLI & API GROUNDING:

```bash
# Amazon S3
aws s3api create-bucket

# CloudFront
aws cloudfront create-distribution

# Global Accelerator
aws globalaccelerator create-accelerator
```

#### PRICING & TCO:

- **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**:
  - Storage cost varies by storage class.
  - Data transfer costs for requests and downloads.

- **[[00_Master_Links_and_Intro|CloudFront]]**:
  - Per GB of data transferred to users.
  - Additional fees for [[Lambda@Edge]] usage, real-time [[vpc-flow-logs|logging]].

- **Global Accelerator**:
  - Monthly charges per endpoint group.
  - Usage-based pricing on actual bytes in and out.

#### COMPETITOR COMPARISON:

- **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] vs Azure Blob Storage**: [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] offers more [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]], better integration with AWS services.
- **[[00_Master_Links_and_Intro|CloudFront]] vs Akamai**: [[cloudfront]] is tightly integrated with AWS services; [[Akamai]] has a broader global network.
- **Global Accelerator vs Google Cloud CDN**: [[Global Accelerator]] provides direct connection to AWS regions for lower latency.

#### INTEGRATION:

- **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**: Integrates seamlessly with [[iam]], [[VPC endpoints]], and [[cloudtrail]].
- **[[00_Master_Links_and_Intro|CloudFront]]**: Works with [[00_Master_Links_and_Intro|IAM]] roles, integrates with [[cloudwatch]] for monitoring.
- **Global Accelerator**: Uses [[00_Master_Links_and_Intro|IAM]] roles; integrates with [[cloudwatch]] for metrics.

#### USE CASES:

1. **E-commerce Website**:
   - Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for static assets, [[00_Master_Links_and_Intro|CloudFront]] for content delivery, and Global Accelerator to optimize global access.

2. **Media Streaming Service**:
   - Store media files in [[S3 Glacier Deep Archive]].
   - Use [[00_Master_Links_and_Intro|CloudFront]] with [[Lambda@Edge]] to handle dynamic content and encryption.

#### DIAGRAMS:

- Placeholder for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] architecture diagram (trade-offs: cost vs. performance).
- Placeholder for [[00_Master_Links_and_Intro|CloudFront]] edge locations map.
- Placeholder for Global Accelerator network layout.

#### EXAM TRAPS:

1. **Misunderstanding Consistency Models**: Differentiate between eventual and read-after-write consistency in [[AWS_SA_PRO_Obsidian_Notes/Master/S3]].
2. **Endpoint Limits**: Be aware of the maximum number of endpoints per accelerator.
3. **Cost Calculations**: Understand pricing tiers and data transfer costs accurately.

#### DECISION MATRIX:

| Features/Use Cases       | E-commerce Website           | Media Streaming Service         |
|--------------------------|------------------------------|---------------------------------|
| [[Master/Srinivas_Notes/S3|S3]]                       | Static asset storage         | Deep Archive for media files    |
| [[00_Master_Links_and_Intro|CloudFront]]               | Content delivery             | Dynamic content manipulation    |
| Global Accelerator       | Global access optimization  | Latency reduction              |

#### EXAM SCENARIOS:

1. **Scenario**: Design a global media streaming service.
   - Solution: Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] for storage, [[cloudfront]] with [[Lambda@Edge]] for dynamic content, and [[Global Accelerator]] for low latency.

2. **Scenario**: Optimize costs for an e-commerce site.
   - Solution: Implement lifecycle [[policies]] in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], use [[00_Master_Links_and_Intro|CloudFront]] [[api-gateway|caching]], monitor with [[cloudwatch]].

#### INTERACTION PATTERNS:

- [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] → [[00_Master_Links_and_Intro|CloudFront]] (content delivery)
- [[00_Master_Links_and_Intro|CloudFront]] → Global Accelerator (traffic optimization)

---

### Audit for [[Global Storage & Content Delivery]]

#### Distractor Analysis

1. **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] vs. Amazon [[00_Master_Links_and_Intro|EFS]]**
   - **Scenario**: An application requires shared file access across multiple [[00_Master_Links_and_Intro|EC2]] instances.
   - > [!danger] Distractor: While [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] provides durable object storage, it's not designed for shared filesystem-like operations. For this scenario, [[Elastic File System (EFS)]] would be more appropriate due to its support for concurrent access and POSIX semantics.

2. **Amazon [[00_Master_Links_and_Intro|CloudFront]] vs. [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]**
   - **Scenario**: You need low latency content delivery via a global network of edge locations.
   > [!danger] Distractor: [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] primarily focuses on accelerating traffic to your applications by directing users to the nearest AWS region, whereas [[cloudfront]] specializes in [[api-gateway|caching]] and delivering static content globally with minimal latency.

3. **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] vs. Amazon [[00_Master_Links_and_Intro|RDS]]**
   - **Scenario**: You need a database service for storing structured data.
   > [!danger] Distractor: [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] is an object storage service, not designed to handle transactional or relational workloads typical of databases. For this scenario, [[rds]] would be more appropriate due to its support for relational databases.

4. **Amazon [[00_Master_Links_and_Intro|EBS]] vs. Amazon [[fsx]]**
   - **Scenario**: You need a high-performance file system for workloads like HPC.
   > [!danger] Distractor: While [[ebs]] provides block-level storage, it doesn't offer the same level of performance and features as [[fsx]], which supports various filesystems (e.g., Lustre, Windows File Server) optimized for specific use cases.

5. **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[00_Master_Links_and_Intro|Glacier]] vs. Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard**
   - **Scenario**: You need fast access to frequently accessed data.
   > [!danger] Distractor: [[S3 Glacier]] is designed for long-term backup and archival storage with retrieval times ranging from minutes to hours, making it unsuitable for frequent or immediate access scenarios.

#### The "Most Correct" Logic: Trade-offs

- **Performance vs. Cost**:
  - **Amazon [[00_Master_Links_and_Intro|CloudFront]]**: Offers high performance by [[api-gateway|caching]] content at edge locations worldwide but incurs costs based on data transfer and requests.
  - **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**: Provides durable storage with varying access patterns (e.g., Standard, Infrequent Access) to balance cost and retrieval time.

- **[[appsync|Security]] vs. Complexity**:
  - **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] with Server-Side Encryption**: Enhances [[appsync|security]] but adds complexity in managing keys through [[AWS KMS]].
  - **[[00_Master_Links_and_Intro|CloudFront]] w/ Signed URLs/Cookies**: Ensures secure content delivery by controlling access, increasing configuration overhead.

#### Enterprise Governance

1. **[[organizations|AWS Organizations]]**:
   - Use service control [[policies]] (SCPs) to restrict services or actions across accounts.
   
2. **Service Control [[policies]] (SCPs)**:
   - Apply SCPs to deny access to sensitive operations, such as disabling [[00_Master_Links_and_Intro|CloudTrail]] [[vpc-flow-logs|logging]] in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets.

3. **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**:
   - Share [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] buckets or [[cloudfront]] distributions securely between AWS accounts within an organization.
  
4. **AWS [[00_Master_Links_and_Intro|CloudTrail]]**:
   - Monitor and audit API activity for all users, roles, and services across your AWS environment to maintain compliance.

#### Tier-3 Troubleshooting

1. **[[kms]] Key Policy Conflicts**:
   > [!warning] Quota: Ensure [[kms]] key [[policies]] allow the necessary permissions for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket encryption.
   - Resolve issues by reviewing [[00_Master_Links_and_Intro|IAM]] role trust relationships and key policy statements to ensure they align with intended access control.

2. **[[00_Master_Links_and_Intro|CloudFront]] Distribution Issues**:
   > [!check] Implementation: Debug distribution configurations (e.g., origin settings, [[api-gateway|caching]] behaviors) using [[cloudwatch|CloudWatch logs]] and [[trusted-advisor|AWS Trusted Advisor]] checks.
   - Investigate edge location performance bottlenecks through monitoring tools like [[Amazon Route 53]] [[route53|health checks]].

#### DECISION MATRIX: Use Case vs. Solution

| Use Case | Recommended Solution |
|----------|----------------------|
| Global Static Content Delivery | Amazon [[00_Master_Links_and_Intro|CloudFront]] |
| Frequent Access to Stored Objects | [[Master/Srinivas_Notes/S3|S3]] Standard Storage Class |
| Infrequent Access to Stored Objects | [[Master/Srinivas_Notes/S3|S3]] Intelligent-Tiering or [[Master/Srinivas_Notes/S3|S3]] Infrequent Access (IA) |
| High-Performance File System for HPC | [[Amazon FSx for Lustre]] |
| Secure, Shared Filesystem across Multiple Instances | Amazon [[00_Master_Links_and_Intro|EFS]] |

#### Recent Updates

1. **2026 GA Features**:
   - Expected new features in [[00_Master_Links_and_Intro|CloudFront]] might include advanced [[appsync|security]] capabilities and enhanced [[api-gateway|caching]] [[policies]].
   
2. **Deprecated Items**:
   - Legacy [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] or API endpoints may be deprecated, requiring migration to newer alternatives (e.g., from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard-Infrequent Access (SIA) to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering).