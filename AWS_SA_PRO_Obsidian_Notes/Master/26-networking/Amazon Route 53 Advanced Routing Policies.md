```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Amazon Route 53 Advanced Routing Policies

#### UNDER-THE-HOOD MECHANICS:
[[Amazon Route 53]] uses [[DNS]] (Domain Name System) to [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Translate|translate]] domain names into IP addresses. The service supports several routing [[policies]], including simple, weighted, latency-based, geolocation, and [[route53|failover routing]]. These [[policies]] enable advanced traffic distribution.

- **Consistency Models**: [[00_Master_Links_and_Intro|Route 53]] ensures eventual consistency across all its edge locations globally.
- **Underlying Infrastructure Logic**: DNS queries are resolved through a distributed network of servers that cache responses to improve performance and reduce load on authoritative servers.

#### EXHAUSTIVE FEATURE LIST:
1. **[[route53|Simple Routing]] Policy**: Routes traffic to one resource or multiple resources with equal weight.
2. **[[route53|Weighted Routing]] Policy**: Distributes traffic based on predefined weights.
3. **[[route53|Latency-Based Routing]] Policy**: Routes requests to the endpoint with the least latency.
4. **[[route53|Geolocation Routing]] Policy**: Directs traffic based on the geographic location of clients.
5. **Geo-Proximity Routing Policy**: Similar to geolocation but includes a factor for proximity to other resources.
6. **[[route53|Failover Routing]] Policy**: Ensures failover to secondary instances if primary instances are unavailable.
7. **Multivalue Answer Routing Policy**: Returns multiple IP addresses to improve performance and availability.

#### STEP-BY-STEP IMPLEMENTATION:
1. Log in to the [[AWS Management Console]] and navigate to [[00_Master_Links_and_Intro|Route 53]].
2. Create a hosted zone for your domain.
3. Define resource record sets with advanced routing [[policies]] (e.g., weighted, latency-based).
4. Configure individual records with the desired settings (weights, [[route53|health checks]], etc.).
5. Verify DNS propagation using tools like `dig` or `nslookup`.

#### TECHNICAL LIMITS:
- **Soft Quotas**:
  - Maximum number of [[route53|hosted zones]] per account: 1000
  - Maximum number of resource record sets per hosted zone: 1 million
- **Hard Quotas**:
  - Number of [[route53|health checks]] per hosted zone: 10,000 (2026 update)
  - Maximum number of [[route53|weighted routing]] [[policies]] per domain: 30

#### CLI & API GROUNDING:
Use the following AWS CLI commands to manage [[00_Master_Links_and_Intro|Route 53]]:

- `aws route53 list-hosted-zones`: Lists all [[route53|hosted zones]].
- `aws route53 change-resource-record-sets`: Creates, updates, or deletes records in a specified hosted zone.

Example command for creating a weighted record set:
```sh
aws route53 change-resource-record-sets --hosted-zone-id Z1234567890 \
--change-batch file://changes.json
```
Where `changes.json` contains the configuration:
```json
{
  "Comment": "Create or update a weighted record set",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "example.com.",
        "Type": "A",
        "Weight": 10,
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "192.0.2.1" }
        ]
      }
    }
  ]
}
```

#### PRICING & TCO:
- **Cost Drivers**: Number of [[route53|hosted zones]], resource record sets, [[route53|health checks]].
- **Optimization Strategies**:
  - Use [[00_Master_Links_and_Intro|Route 53]] DNS failover to avoid additional costs for redundant services.
  - Utilize TTL (Time To Live) settings to minimize unnecessary queries.

#### COMPETITOR COMPARISON:
- **AWS [[00_Master_Links_and_Intro|Route 53]] vs. Google Cloud DNS**: Both offer advanced routing [[policies]], but [[00_Master_Links_and_Intro|Route 53]] has better global coverage and more health check options.
- **[[00_Master_Links_and_Intro|Route 53]] vs. Azure DNS**: [[00_Master_Links_and_Intro|Route 53]] offers a broader range of routing types and is integrated with AWS services.

#### INTEGRATION:
[[Route 53]] integrates seamlessly with [[00_Master_Links_and_Intro|other AWS services]]:

- **[[cloudwatch]]**: For monitoring DNS query metrics and [[route53|health checks]].
- **[[00_Master_Links_and_Intro|IAM]]**: For managing permissions to manage [[route53|hosted zones]] and records.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: [[00_Master_Links_and_Intro|Route 53]] can be used for [[route53|private hosted zones]] within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

#### USE CASES:
1. **E-commerce Website with Multiple Data Centers**: Use [[route53|latency-based routing]] to send traffic to the nearest data center.
2. **Global Content Delivery Network (CDN)**: Implement [[route53|geolocation routing]] to serve content from the closest CDN edge location.

#### DIAGRAMS:
- **[[route53|Simple Routing]] Policy Diagram**: Placeholder for a diagram showing a single A record pointing to an IP address.
- **[[route53|Weighted Routing]] Policy Diagram**: Placeholder for a diagram illustrating multiple records with different weights.

> [!abstract] Exam Tip
Misunderstanding the difference between latency-based and geo-proximity routing [[policies]], confusing [[route53|hosted zones]] with resource record sets, or not understanding how [[route53|health checks]] interact with [[route53|failover routing]] are common mistakes on the exam. Be prepared to identify these traps accurately.

#### DECISION MATRIX:
| Feature              | Use Case                    |
|----------------------|-----------------------------|
| [[route53|Simple Routing]]       | Small websites, low traffic |
| [[route53|Weighted Routing]]     | Load balancing between servers|
| Latency-Based        | Global service with multiple regions |
| Geolocation          | Content localization based on region |

#### 2026 UPDATES:
- Enhanced [[appsync|security]] features including DNSSEC support.
- Improved performance analytics through [[cloudwatch]] integration.

#### EXAM SCENARIOS:
1. **Scenario**: You need to ensure high availability for a global application with multiple data centers.
   - **Solution**: Use [[route53|latency-based routing]] combined with [[route53|health checks]].

2. **Scenario**: Implementing failover for a mission-critical service.
   - **Solution**: Configure [[route53|failover routing]] policy and set up [[route53|health checks]] for monitoring.

#### INTERACTION PATTERNS:
- [[00_Master_Links_and_Intro|Route 53]] interacts with [[cloudwatch]] to send metrics about DNS queries and [[route53|health checks]].
- It integrates with VPCs via [[route53|private hosted zones]], which are accessible only within the specified [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

### Audit of Draft on Amazon Route 53 Advanced Routing Policies

#### Distractor Analysis:
**Scenarios where Amazon [[00_Master_Links_and_Intro|Route 53]] is a "wrong" answer:**
- **Load Balancing with [[00_Master_Links_and_Intro|EC2]] Instances:** While [[00_Master_Links_and_Intro|Route 53]] can route traffic to different resources, it isn't designed for load balancing across multiple [[00_Master_Links_and_Intro|EC2]] instances directly; instead, using Elastic Load Balancer ([[elb]]) would be more appropriate.
- **Dedicated Content Delivery Network (CDN):** Amazon [[00_Master_Links_and_Intro|CloudFront]] is a dedicated CDN service that should be used when the primary need is content delivery and [[api-gateway|caching]] rather than DNS routing.
- **High-Frequency Data Queries:** [[00_Master_Links_and_Intro|Route 53]] is not optimized for high-frequency data queries or real-time application traffic; instead, consider using [[dynamodb]] or another database service.
- **Dynamic Hostname Resolution in Private Networks:** For private network hostname resolution, services like [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] and internal [[00_Master_Links_and_Intro|Route 53]] Resolver would be more suitable.
- **Advanced Application-Level Load Balancing:** Advanced load balancing needs that require sophisticated session persistence and [[route53|health checks]] should use [[elb]] or [[00_Master_Links_and_Intro|Application Load Balancer (ALB)]].

> [!danger] Distractor
Distinguishing between using [[00_Master_Links_and_Intro|Route 53]] for DNS routing versus services like [[cloudfront]], [[elb]], or [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink]] is crucial.

#### The "Most Correct" Logic:
**Trade-offs: Performance vs. Cost**
- **Performance:** [[00_Master_Links_and_Intro|Route 53]] provides low-latency DNS resolution, enhancing application performance by directing traffic to the nearest server geographically.
- **Cost:** Advanced routing [[policies]] can add complexity and cost due to additional configurations (e.g., [[route53|Latency-based routing]] might require setting up multiple endpoints in various regions). However, this trade-off is often justified for applications requiring high availability and performance.

> [!warning] Quota
Be aware of the soft and hard quotas related to [[route53|hosted zones]] and [[route53|health checks]].

#### Enterprise Governance:
**[[organizations|AWS Organizations]], SCPs, [[ram]], and [[00_Master_Links_and_Intro|CloudTrail]] Integration:**
- **[[organizations|AWS Organizations]]:** Use [[organizations|AWS Organizations]] to manage [[00_Master_Links_and_Intro|Route 53]] [[policies]] across multiple accounts. This allows centralized management of DNS configurations.
- **Service Control [[policies]] (SCPs):** Implement SCPs to restrict access to sensitive [[00_Master_Links_and_Intro|Route 53]] operations, ensuring only authorized users can modify DNS settings.
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Utilize [[ram]] to share [[route53|hosted zones]] and private [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] associations across different AWS accounts within your organization.
- **[[00_Master_Links_and_Intro|CloudTrail]]:** Enable [[00_Master_Links_and_Intro|CloudTrail]] [[vpc-flow-logs|logging]] for [[00_Master_Links_and_Intro|Route 53]] actions to monitor who performed what action in the service. This helps in auditing and compliance.

#### Tier-3 Troubleshooting:
**Complex Failure Modes:**
- **[[kms]] Key Policy Conflicts:** If you're using [[kms]] encryption keys to secure your DNS records, ensure that [[00_Master_Links_and_Intro|IAM]] roles have appropriate permissions to access these keys. Misconfigured [[policies]] can lead to failures where [[00_Master_Links_and_Intro|Route 53]] is unable to perform certain actions.
- **[[route53|Health Checks]] Failures:** Ensure that health check configurations are correctly set up and match the expected response from endpoints. Misconfigured [[route53|health checks]] can cause unexpected routing behavior or downtime.
- **[[route53|Latency-Based Routing]] Issues:** Incorrectly configured [[route53|latency-based routing]] [[policies]] may result in suboptimal traffic distribution, leading to increased latency.

#### Decision Matrix:
**Use Case vs. Solution Table:**

| Use Case                      | Recommended Solution               |
|-------------------------------|------------------------------------|
| Geographically Distributed App| [[route53|Latency-Based Routing]] Policy       |
| High Availability             | Failover and Weighted [[policies]]     |
| Secure DNS Resolution         | [[route53|Private Hosted Zones]] and [[Master/Srinivas_Notes/VPC|VPC]] Link  |
| Scalable Load Balancing       | [[00_Master_Links_and_Intro|Route 53]] with [[elb]] or [[ALB]]           |

#### Recent Updates:
**GA Features and Deprecated Items for 2026:**
- **New GA Features:** Look out for any new advanced routing [[policies]] or integration updates announced by AWS.
- **Deprecated Items:** [[nonportable_instructions|Review]] AWS documentation to identify any services, features, or APIs that will be deprecated in 2026.

---

This audit provides a comprehensive [[nonportable_instructions|review]] of Amazon Route 53 Advanced Routing Policies, covering distractor scenarios, performance trade-offs, governance practices, complex troubleshooting issues, decision-making tables, and upcoming updates.