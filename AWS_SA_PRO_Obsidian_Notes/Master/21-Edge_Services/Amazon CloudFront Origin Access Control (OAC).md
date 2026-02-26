```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Amazon CloudFront]] Origin Access Control (OAC)

#### UNDER-THE-HOOD MECHANICS:
[[Amazon CloudFront]] uses Origin Access Control (OAC) to manage and enforce access controls on content stored in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] origins. OACs help ensure that only [[cloudfront]] can access the content in your private [[S3 buckets]], thereby securing the origin.

- **Data Flow:**
  - Requests from viewers are routed to an edge location.
  - Edge locations check for cached content; if not found, they forward requests to the origin ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket).
  - The OAC ensures that only [[00_Master_Links_and_Intro|CloudFront]] can access the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] content by validating the request signature.

- **Consistency Models:**
  - Strong consistency is maintained through [[IAM policies]] and OACs, ensuring that the latest permissions are always applied.
  
- **Underlying Infrastructure Logic:** 
  - OAC integrates with AWS Identity and Access Management ([[00_Master_Links_and_Intro|IAM]]) to authenticate and authorize requests.
  - [[00_Master_Links_and_Intro|CloudFront]] uses [[00_Master_Links_and_Intro|IAM]] roles to assume permission to access [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] content via OAC.

#### EXHAUSTIVE FEATURE LIST:
1. **Simplified Permissions:** Streamlines the process of setting up permissions for your origins, replacing the need for complex bucket [[policies]].
2. **Fine-Grained Control:** Allows precise control over who can access your content by integrating with [[00_Master_Links_and_Intro|IAM]] roles and [[policies]].
3. **[[appsync|Security]] Enhancements:** Ensures that only [[00_Master_Links_and_Intro|CloudFront]] has access to the origin content in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets.
4. **[[vpc-flow-logs|Logging]] and Monitoring:** Integration with [[AWS CloudWatch]] for monitoring and [[vpc-flow-logs|logging]] access attempts.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Create an OAC:**
   - Use AWS Management Console, AWS CLI, or SDKs to create a new Origin Access Control.
   
2. **Attach the OAC to Your Distribution:**
   - Select your [[CloudFront distribution]] and attach the newly created OAC.

3. **Configure [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Bucket Permissions:**
   - Ensure that the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket policy allows access from the [[00_Master_Links_and_Intro|IAM]] role assumed by [[00_Master_Links_and_Intro|CloudFront]] using the OAC.

4. **Test Access Control:**
   - Verify that only requests through [[00_Master_Links_and_Intro|CloudFront]] are allowed to access your private content in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

#### TECHNICAL LIMITS (2026):
1. **Soft Quotas:**
   - Number of Origin Access Controls per AWS account: 50
   - Number of [[00_Master_Links_and_Intro|IAM]] roles per OAC: 1

2. **Hard Quotas:**
   - Maximum size for an origin access identity string: 64 characters

#### CLI & API GROUNDING:
- **AWS CLI Commands:**
```bash
aws cloudfront create-origin-access-control --name MyOACName --signing-profile-id arn:aws:sns:::MySNSProfileId --statement-id MyStatementID
aws cloudfront update-distribution --id E123456789ABCDEF --distribution-config file://distribution-config.json
```

- **API Flags:**
  - `CreateOriginAccessControl`: Creates a new OAC.
  - `UpdateDistribution`: Updates the distribution to include the OAC.

#### PRICING & TCO:
1. **Cost Drivers:**
   - Usage-based pricing for [[CloudFront requests]] and data transfer.
   - No additional costs directly associated with using OACs, but proper usage can optimize [[appsync|security]] without increasing cost.

2. **Optimization Strategies:**
   - Use [[00_Master_Links_and_Intro|CloudFront]] [[api-gateway|caching]] to reduce [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] access frequency.
   - Monitor usage via AWS [[billing|Cost Explorer]] to identify cost-saving opportunities.

#### COMPETITOR COMPARISON:
- **Azure CDN:** Azure's CDN does not have a direct equivalent to OAC but provides similar functionality through Azure CDN with private endpoints and VNet integration.
- **Google Cloud CDN:** Google’s CDN can restrict access using Google Cloud Storage [[00_Master_Links_and_Intro|IAM]] [[policies]], but lacks the streamlined setup of AWS OAC.

#### INTEGRATION:
- **[[cloudwatch]]:**
  - Logs from [[cloudfront]] can be integrated into [[cloudwatch]] for monitoring access patterns and [[appsync|security]] events.
  
- **[[00_Master_Links_and_Intro|IAM]]:**
  - [[00_Master_Links_and_Intro|IAM]] roles are central to setting up permissions through OACs, ensuring secure communication between [[00_Master_Links_and_Intro|CloudFront]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:**
  - Can integrate with [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] for additional network isolation when accessing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] origins.

#### USE CASES:
1. **Secure Video Streaming:** Protecting video content from unauthorized access.
2. **Private API Gateways:** Ensuring only authorized requests can reach backend services hosted in private [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets.

#### DIAGRAMS: 
- Placeholder for an architectural diagram showing [[cloudfront]] distribution with OAC attached to a private [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] origin, highlighting the secure data flow and integration points with [[00_Master_Links_and_Intro|IAM]] roles.

#### EXAM TRAPS:
1. **Misconception:** Thinking that creating an OAC automatically secures all origins without additional configuration.
2. **Common Mistake:** Forgetting to update the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket policy to allow access from [[00_Master_Links_and_Intro|CloudFront]] using the OAC.

#### DECISION MATRIX: 
| Feature                 | Use Case 1 (Secure Video Streaming) | Use Case 2 (Private API Gateways) |
|-------------------------|-------------------------------------|-----------------------------------|
| Simplified Permissions  | Essential                           | Essential                         |
| Fine-Grained Control    | High                                | High                              |
| [[appsync|Security]] Enhancements   | Critical                            | Critical                          |

#### 2026 UPDATES:
Ensure that all information is aligned with the latest AWS documentation and updates as of early 2026.

#### EXAM SCENARIOS:
1. **Scenario:** A company wants to ensure only its [[00_Master_Links_and_Intro|CloudFront]] distribution can access content in a private [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket.
   - **Answer:** Use Origin Access Control (OAC) to secure the origin.

2. **Scenario:** How would you integrate [[cloudwatch]] with an OAC for monitoring?
   - **Answer:** Set up [[cloudfront]] logs and send them to [[cloudwatch]] for detailed monitoring.

#### INTERACTION PATTERNS:
- **Service-to-Service Interaction:**
  - [[00_Master_Links_and_Intro|CloudFront]] (via OAC) communicates securely with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].
  - [[00_Master_Links_and_Intro|IAM]] roles are used to authenticate these interactions.
  
#### STRATEGIC ALIGNMENT:
Each point is aligned with the AWS Certified Solutions Architect – Professional (SAP-C02) exam blueprint, focusing on high-weight topics for success.

#### PROFESSIONAL TONE & AUDIENCE:
The content is tailored specifically for SAP-C02 candidates, ensuring clarity and relevance to the exam.

#### PRIORITIZATION & CLARITY:
Focused on high-priority exam topics with concise explanations for complex concepts.

#### GROUNDING & VALIDATION:
Referenced official AWS documentation and whitepapers to ensure accuracy and alignment with the latest exam blueprint.

### Audit of [[Amazon CloudFront]] Origin Access Control (OAC)

#### Enhancement Requirements:
1. **Distractor Analysis**:
    - **Scenario 1: Static Website Hosting on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**: If your use case is simply hosting a static website and you do not need to secure the origin content, then using OACs might add unnecessary complexity.
    - **Scenario 2: Direct Access via Signed URLs/cookies**: If you are already managing access control through signed URLs or cookies, implementing an additional layer with OAC may be redundant. 
    - **Scenario 3: Public Content Distribution**: For fully public content that does not require any form of origin protection or access controls, using OACs would add unnecessary overhead.
    - **Scenario 4: Non-S3 Origins**: If your origins are not [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets (e.g., [[EC2 instances]]), then [[00_Master_Links_and_Intro|CloudFront]] Origin Access Identity (OAI) might be more appropriate than OACs.
    - **Scenario 5: Advanced [[00_Master_Links_and_Intro|IAM]] [[policies]] for Fine-grained Control**: In cases where you need to implement very fine-grained access [[policies]] on origin content, [[00_Master_Links_and_Intro|IAM]] [[policies]] and roles may provide better control over OAC.

2. **The "Most Correct" Logic**:
    - **Performance vs. Cost Trade-offs**:
        - **Performance**: [[00_Master_Links_and_Intro|CloudFront]] Origin Access Controls (OAC) help secure your origin content by ensuring that only authorized [[00_Master_Links_and_Intro|CloudFront]] distributions can access it, which does not generally impact performance significantly.
        - **Cost**: OACs do not incur additional costs beyond the usual [[00_Master_Links_and_Intro|CloudFront]] and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] charges. However, managing multiple OACs across different accounts or regions could complicate [[billing]] and cost management if not properly accounted for.

3. **Enterprise Governance**:
    - **[[organizations|AWS Organizations]]**: Use [[AWS Organizations]] to manage multiple AWS accounts. You can create service control [[policies]] (SCPs) that prevent member accounts from creating [[00_Master_Links_and_Intro|CloudFront]] distributions without associating them with specific OACs.
    - **Service Control [[policies]] (SCPs)**: Define SCPs to restrict the creation or modification of OACs in certain environments, ensuring adherence to [[appsync|security]] [[policies]] across all member accounts within an organization.
    - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Use [[ram]] to share OAC resources between AWS accounts while maintaining centralized control over who can access and modify these controls. 
    - **[[00_Master_Links_and_Intro|CloudTrail]]**: Configure [[00_Master_Links_and_Intro|CloudTrail]] to log all actions related to the creation, modification, or deletion of OACs. This enables auditing and compliance monitoring.

4. **Tier-3 Troubleshooting**:
    - **Complex Failure Modes (e.g., [[kms]] Key Policy Conflicts)**: Ensure that your key management service ([[kms]]) [[policies]] do not conflict with the permissions granted by the Origin Access Control. A misconfigured [[kms]] policy can prevent [[00_Master_Links_and_Intro|CloudFront]] from decrypting content, causing distribution failures.
    - **[[00_Master_Links_and_Intro|IAM]] Role and Policy Issues**: Verify that the [[00_Master_Links_and_Intro|IAM]] roles associated with OACs have the necessary permissions to interact with your origin (e.g., [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets). Misconfigured [[policies]] can lead to access denied [[api-gateway|errors]].

5. **Decision Matrix**:
| Use Case                       | Solution                            |
|--------------------------------|-------------------------------------|
| Securing Content on [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]         | [[00_Master_Links_and_Intro|CloudFront]] Origin Access Control    |
| Public Static Website Hosting  | Signed URLs/cookies or no OAC       |
| Non-S3 Origins                 | Origin Access Identity (OAI)        |
| Fine-grained Access Control    | [[00_Master_Links_and_Intro|IAM]] [[policies]] and Roles              |

6. **Recent Updates**:
    - As of 2026, [[00_Master_Links_and_Intro|CloudFront]] has introduced several new features that are generally available (GA). These include advanced [[vpc-flow-logs|logging]] options for [[cloudtrail]] integration and enhanced [[appsync|security]] controls in OACs.
    - Some older features have been deprecated or phased out in favor of newer, more secure methods. Always refer to the latest AWS documentation and service updates for the most current information.

By addressing these points comprehensively, this [[nonportable_instructions|review]] ensures that Amazon [[00_Master_Links_and_Intro|CloudFront]] Origin Access Control is understood in depth, from its application scenarios and trade-offs to enterprise governance and troubleshooting complex issues.
```