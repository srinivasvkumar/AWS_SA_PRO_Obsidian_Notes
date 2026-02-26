```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
priority: high
```

## Enhanced Deep-Dive Technical Overview: Amazon [[00_Master_Links_and_Intro|CloudFront]] vs. [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]

### Under-the-Hood Mechanics

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
- **Internal Data Flow**: [[00_Master_Links_and_Intro|CloudFront]] uses a global network of edge locations to cache content from origins (such as [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets or [[00_Master_Links_and_Intro|EC2]] instances). Requests are routed through the nearest edge location to reduce latency.
- **Consistency Models**: Supports different cache invalidation methods, including `InvalidateObjects` for specific files and `CreateInvalidation` for entire paths.
- **Underlying Infrastructure Logic**: Uses DNS routing [[policies]] (e.g., geo-based or [[route53|weighted routing]]) to distribute requests across edge locations. Integrates with AWS WAF for [[appsync|security]].

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- **Internal Data Flow**: Utilizes anycast IP addresses and directs traffic to the optimal endpoint based on latency, [[route53|health checks]], and client location.
- **Consistency Models**: Does not cache content; focuses solely on directing traffic efficiently. [[route53|Health checks]] ensure only healthy endpoints receive traffic.
- **Underlying Infrastructure Logic**: Uses a network of edge locations that route requests via anycast IP addresses to the nearest region or endpoint based on optimal performance.

### Exhaustive Feature List

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
- **[[api-gateway|Caching]] & Edge Locations**: Caches content at over 150 locations worldwide.
- **Custom Headers & Cookies Management**: Allows customization and manipulation of headers and cookies for [[api-gateway|caching]] purposes.
- **[[Lambda@Edge]] Integration**: Executes custom code ([[00_Master_Links_and_Intro|lambda functions]]) in response to viewer or origin requests/events.
- **[[appsync|Security]] with AWS WAF**: Integrates with AWS WAF for DDoS protection, IP whitelisting/blacklisting, etc.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- **Anycast IPs**: Uses anycast to direct traffic from the client to the optimal region/endpoint.
- **[[route53|Health Checks]] & Routing Control**: Performs continuous [[route53|health checks]] and routes traffic only to healthy endpoints.
- **Cross-Region Failover**: Provides automatic failover between regions if an endpoint becomes unhealthy.
- **Custom DNS Names**: Allows use of custom domain names for anycast IP addresses.

### Step-by-Step Implementation

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
1. **Create a Distribution**: Define origin ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket or [[00_Master_Links_and_Intro|EC2]] instance) and configure cache behaviors.
   ```json
   {
     "DistributionConfig": {
       "Origins": {
         "Quantity": 1,
         "Items": [
           {
             "Id": "MyOrigin",
             "DomainName": "my-bucket.s3.amazonaws.com"
           }
         ]
       },
       "CacheBehaviors": {
         "Quantity": 1,
         "Items": [
           {
             "PathPattern": "/static/*",
             "TargetOriginId": "MyOrigin",
             "ForwardedValues": {
               "QueryString": true
             }
           }
         ]
       }
     }
   }
   ```
2. **Configure [[api-gateway|Caching]] Rules**: Set TTLs, query string parameters to include/exclude from [[api-gateway|caching]].
   ```json
   {
     "CacheBehaviors": {
       "Quantity": 1,
       "Items": [
         {
           "PathPattern": "/static/*",
           "TargetOriginId": "MyOrigin",
           "ForwardedValues": {
             "QueryString": true
           },
           "MinTTL": 3600,
           "DefaultTTL": 86400,
           "MaxTTL": 31536000
         }
       ]
     }
   }
   ```
3. **Integrate with AWS WAF**: Create Web ACL and associate it with the [[00_Master_Links_and_Intro|CloudFront]] distribution for [[appsync|security]].
4. **[[Lambda@Edge]] Integration**: Write and upload [[00_Master_Links_and_Intro|Lambda functions]] to run at different points in request/response cycle.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
1. **Create an Accelerator**: Define endpoints (e.g., [[00_Master_Links_and_Intro|EC2]] instances, Application Load Balancers).
   ```json
   {
     "Name": "MyAccelerator",
     "Enabled": true,
     "IpAddressType": "IPV4"
   }
   ```
2. **Set Up Endpoint Groups**: Configure groups of endpoints for each region.
   ```json
   {
     "EndpointGroupRegion": "us-west-2",
     "EndpointConfigurations": [
       {
         "Id": "MyEndpoint1",
         "Weight": 50,
         "EndpointId": "my-alb-us-west-2"
       }
     ]
   }
   ```
3. **Enable [[route53|Health Checks]]**: Specify health check settings to monitor endpoint status.
4. **Configure Routing Control**: Set up failover [[policies]] and routing control.

### Technical Limits

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
- **Distributions per account**: 200 (adjustable via support request).
- **Cache Invalidation Requests per distribution**: 1,000 invalidation requests per month free; additional cost for more.
- **Origin Access Identities (OAI)**: Up to 50 OAI can be created.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- **Accelerators per account**: 25 by default (adjustable via support request).
- **Endpoints per accelerator**: 32.
- **Endpoint groups per region**: 4 (max 16 across all regions).

### CLI & API Grounding

#### Amazon [[00_Master_Links_and_Intro|CloudFront]] CLI/API
```sh
aws cloudfront create-distribution --distribution-config <path-to-config>
aws cloudfront create-invalidation --distribution-id <id> --invalidation-batch <path-to-batch>
```

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]] CLI/API
```sh
aws globalaccelerator create-accelerator --name my-accelerator --ip-address-type IPV4
aws globalaccelerator create-endpoint-group --listener-arn <listener-arn> --endpoint-group-config <config-file>
```

### Pricing & TCO

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
- **Pricing**: Pay per GB of data transferred out, plus costs for [[Lambda@Edge]] and WAF usage.
- **Optimization Strategies**: Use cache effectively, leverage [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration for large file uploads/downloads.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- **Pricing**: Per-region charges based on the number of accelerator endpoints used. Additional costs for high traffic volumes.
- **Optimization Strategies**: Ensure [[route53|health checks]] are configured correctly to avoid unnecessary routing changes.

### Competitor Comparison

#### Amazon [[00_Master_Links_and_Intro|CloudFront]] vs. [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- **Use Case Fit**: [[00_Master_Links_and_Intro|CloudFront]] is ideal for content delivery with [[api-gateway|caching]], while Global Accelerator is designed for low-latency routing without [[api-gateway|caching]].
- **Trade-offs**: [[00_Master_Links_and_Intro|CloudFront]] offers [[api-gateway|caching]] and [[Lambda@Edge]] capabilities but requires more setup. Global Accelerator provides efficient routing but doesn’t cache content.

### Contextual Integration

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]
- Integrates well with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[00_Master_Links_and_Intro|EC2]], AWS WAF, and [[Lambda@Edge]] for comprehensive web delivery.
- Fits into patterns like Content Delivery Networks (CDNs) and microservices where [[api-gateway|caching]] is crucial.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]
- Integrates with ALBs, [[00_Master_Links_and_Intro|EC2]] instances, and other network endpoints.
- Fits into architectures requiring low-latency global routing across regions or services.

### Real-World Use Cases

#### Amazon [[00_Master_Links_and_Intro|CloudFront]]: 
- **Case Study**: A large e-commerce site uses [[00_Master_Links_and_Intro|CloudFront]] to cache static assets (images, JavaScript files) at edge locations for faster page load times globally.

#### [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]]: 
- **Case Study**: An enterprise application with multi-region deployment uses Global Accelerator to route client traffic to the nearest healthy endpoint, ensuring minimal latency and high availability.

### Architectural Diagrams

1. **[[00_Master_Links_and_Intro|CloudFront]] Integration Diagram**
   - Edge Location -> Viewer
   - Origin (S3/EC2) <- Edge Location
   - [[Lambda@Edge]] runs at different points in the request/response cycle.
   
2. **Global Accelerator Integration Diagram**
   - Anycast IP -> Client
   - Region A Endpoint Group <-> Global Accelerator
   - Region B Endpoint Group <-> Global Accelerator

### Exam Traps & Misconceptions

#### Common Mistakes 
- Confusing [[00_Master_Links_and_Intro|CloudFront]] and Global Accelerator roles ([[api-gateway|caching]] vs. routing).
- Not understanding cache invalidation methods in [[00_Master_Links_and_Intro|CloudFront]].
- **Distractor Analysis**: 
  - **Scenario 1:** High RPO/RTO requirements: If the requirement is for ultra-low latency and high availability without [[api-gateway|caching]], choosing [[00_Master_Links_and_Intro|CloudFront]] might be incorrect due to its added complexity.
  - **Scenario 2:** [[00_Master_Links_and_Intro|Cost Optimization]]: For purely routing needs without [[api-gateway|caching]], Global Accelerator would be more cost-effective than [[00_Master_Links_and_Intro|CloudFront]].
  - **Scenario 3:** Enterprise Governance: Without proper SCPs or [[00_Master_Links_and_Intro|IAM]] [[policies]], using either service could lead to [[appsync|security]] gaps; ensure [[organizations|AWS Organizations]] and SCPs are configured correctly.

### Architectural Trade-offs

- **Operational Excellence**: Using both services can enhance operational efficiency by leveraging [[00_Master_Links_and_Intro|CloudFront]] for content delivery and Global Accelerator for efficient routing.
- **[[00_Master_Links_and_Intro|Cost Optimization]]**: Consider traffic patterns and [[api-gateway|caching]] requirements to optimize costs. [[00_Master_Links_and_Intro|CloudFront]] might be cost-effective for frequently accessed content, while Global Accelerator is better for low-latency routing.

### Enterprise Governance

#### [[organizations|AWS Organizations]]
- Use SCPs (Service Control [[policies]]) to restrict access and enforce compliance.
- Configure [[00_Master_Links_and_Intro|IAM]] roles and [[policies]] using AWS Identity Center.
- Utilize [[ram]] (Resource Access Manager) to share resources across accounts securely.

#### [[00_Master_Links_and_Intro|CloudTrail]] Auditing
- Enable [[00_Master_Links_and_Intro|CloudTrail]] to log API calls for both services. Monitor logs for unauthorized or suspicious activity.

### Tier-3 Troubleshooting

#### Complex Failure Modes: 
- **[[kms]] Key Policy Conflicts**: When migrating cross-account, ensure [[kms]] keys are correctly configured and accessible by all required accounts.
- **Health Check Issues**: Misconfigured [[route53|health checks]] can lead to routing traffic to unhealthy endpoints. Ensure health check settings match the application’s availability criteria.

### Decision Matrix

| Feature/Implementation       | Amazon [[00_Master_Links_and_Intro|CloudFront]]                      | [[SAP-C02_Vault/AWS Global Accelerator|AWS Global Accelerator]]                 |
|------------------------------|-----------------------------------------|----------------------------------------|
| [[api-gateway|Caching]]                      | Yes, via edge locations                | No [[api-gateway|caching]]                             |
| Routing Control              | Via DNS [[policies]]                       | Anycast IP routing                     |
| [[route53|Health Checks]]                | Basic                                  | Continuous [[route53|health checks]]               |
| Failover                     | Manual invalidation                    | Automatic failover                     |

### Exam Questions & Scenarios

1. **Q**: Which service is better for low-latency global routing without [[api-gateway|caching]]?
   - **A**: [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]].
2. **Q**: How would you ensure that a globally distributed application has minimal latency and high availability using AWS services?
   - **A**: Use [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]] to route traffic efficiently between regions, combined with [[00_Master_Links_and_Intro|CloudFront]] for content delivery if needed.

### Integration Points

#### [[00_Master_Links_and_Intro|CloudFront]]
- Integrates well with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[00_Master_Links_and_Intro|EC2]], WAF, [[Lambda@Edge]].
  
#### Global Accelerator
- Works seamlessly with ALBs, [[00_Master_Links_and_Intro|EC2]] instances across regions.

This overview ensures comprehensive coverage of both services' features, limits, implementation steps, and integration within the AWS ecosystem, tailored to the SAP-C02 exam needs.

### Related [[vpc-flow-logs|Notes]]
- [Amazon S3](AWS_SA_PRO_Obsidian_Notes/Master/S3.md)
- [AWS WAF](WAF.md)
- [Lambda@Edge](LambdaAtEdge.md)
- [Application Load Balancer (ALB)](ALB.md)

---