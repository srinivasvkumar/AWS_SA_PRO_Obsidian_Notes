```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[Enterprise Security & Governance]] in AWS (2026)

#### UNDER-THE-HOOD MECHANICS:
1. **Data Flow**: Data flows through various [[appsync|security]] layers, including [[Identity and Access Management]], network controls via [[VPCs]], encryption services like [[kms]], and compliance checks using [[AWS Config]].
2. **Consistency Models**: [[appsync|Security]] configurations are applied consistently across resources to ensure uniform [[appsync|security]] [[policies]]. [[00_Master_Links_and_Intro|IAM]] roles and [[policies]] define access levels granularly.
3. **Underlying Infrastructure Logic**: The infrastructure uses a combination of software-defined networking ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]), hardware-based encryption (HSMs for [[kms]]), and automated policy enforcement via [[AWS Config]] and [[GuardDuty]].

#### EXHAUSTIVE FEATURE LIST:
1. **[[00_Master_Links_and_Intro|IAM]]**:
   - Identity Management
   - Access Control [[policies]] ([[00_Master_Links_and_Intro|IAM]] [[policies]])
   - Roles, Users, Groups
   - Service-linked roles

2. **Network [[appsync|Security]]**:
   - [[VPCs]]
   - Subnets
   - Network ACLs
   - [[appsync|Security]] Groups
   - [[Transit gateway]]

3. **Encryption Services**:
   - [[kms]] for encryption key management
   - [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] server-side and client-side encryption
   - [[00_Master_Links_and_Intro|EBS]] encryption

4. **Compliance & Auditing**:
   - [[AWS Config]] (configurations monitoring)
   - [[AWS Trusted Advisor]]
   - AWS [[00_Master_Links_and_Intro|CloudTrail]] (activity [[vpc-flow-logs|logging]])

5. **Threat Detection & Response**:
   - Amazon [[GuardDuty]] (threat detection service)
   - [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/inspector|AWS Inspector]] (automated [[appsync|security]] assessments)
   - [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/macie|Amazon Macie]] (data privacy and protection)

6. **Compliance Tools**:
   - [[AWS Artifact]] (compliance documentation)
   - [[AWS Config Rules]]

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set Up [[00_Master_Links_and_Intro|IAM]] [[policies]]**: Define roles, users, groups, and [[policies]].
2. **Configure VPCs & [[appsync|Security]] Groups**: Set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] subnets, network ACLs, and [[appsync|security]] groups.
3. **Enable Encryption Services**: Configure [[kms]] keys for services like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[00_Master_Links_and_Intro|EBS]].
4. **Deploy Compliance Tools**: Enable [[config|AWS Config]] rules and [[00_Master_Links_and_Intro|CloudTrail]] [[vpc-flow-logs|logging]].
5. **Integrate Threat Detection**: Use [[GuardDuty]] to monitor threats in your environment.

#### TECHNICAL LIMITS (2026):
1. **[[00_Master_Links_and_Intro|IAM]]**:
   - 5,000 [[policies]] per account
   - 1,000 groups per account

2. **Network [[appsync|Security]]**:
   - 200 subnets per [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
   - 5 [[appsync|security]] groups per instance

3. **Encryption Services**:
   - 10,000 [[kms]] keys per region
   - Data at rest encryption limits vary by service

4. **Compliance & Auditing**:
   - [[config|AWS Config]] recording limits (e.g., max 2,500 resource configurations recorded)

#### CLI & API GROUNDING:
- **[[00_Master_Links_and_Intro|IAM]]**: `aws iam create-user`, `aws iam attach-user-policy`
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: `aws ec2 describe-subnets`, `aws ec2 authorize-security-group-ingress`
- **[[kms]]**: `aws kms create-key`, `aws kms encrypt`
- **[[00_Master_Links_and_Intro|CloudTrail]]**: `aws cloudtrail put-event-selectors`

#### PRICING & TCO:
1. **[[00_Master_Links_and_Intro|IAM]]**: Free for basic use, fees apply for advanced features.
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: $0.05 per hour for each [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] with a subnet; $3 per month for each active [[00_Master_Links_and_Intro|NAT Gateway]].
3. **[[kms]]**: $1 per region per month plus charges based on key operations.
4. **[[00_Master_Links_and_Intro|CloudTrail]]**: Pricing varies by volume of logs and storage.

#### COMPETITOR COMPARISON:
- **Azure**:
  - Azure AD for [[00_Master_Links_and_Intro|IAM]]
  - Network [[appsync|Security]] Groups (NSGs)
  - Key Vault for encryption

- **GCP**:
  - Google Cloud Identity & Access Management ([[00_Master_Links_and_Intro|IAM]])
  - [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Firewalls
  - [[kms]] for key management

#### INTEGRATION:
1. **[[cloudwatch]]**: Integration with logs and metrics to monitor [[appsync|security]].
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: [[appsync|Security]] Groups and NACLs integrated into network architecture.
3. **[[00_Master_Links_and_Intro|IAM]]**: Roles and [[policies]] integrated across services.

#### USE CASES:
1. **Financial Services**:
   - Compliance monitoring using [[AWS Config]]
   - Encryption for sensitive data

2. **Healthcare**:
   - HIPAA compliance checks with Trusted Advisor
   - Data privacy protection with [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]]

3. **Retail**:
   - Network [[appsync|security]] to prevent DDoS attacks
   - [[00_Master_Links_and_Intro|IAM]] [[policies]] for access control

#### DIAGRAMS:
1. **[[00_Master_Links_and_Intro|IAM]] Architecture**: Placeholder diagram illustrating roles, users, and groups.
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|Security]]**: Diagram showing subnets, NACLs, and SGs.

#### EXAM TRAPS:
- Misunderstanding the number of [[appsync|security]] groups per instance.
- Confusing [[00_Master_Links_and_Intro|IAM]] [[policies]] with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket [[policies]].
- Overlooking the importance of encryption at rest vs. in transit.

> [!abstract] Exam Tip
Misunderstanding the number of [[appsync|security]] groups per instance can lead to configuration [[api-gateway|errors]] and potential [[appsync|security]] gaps.

#### DECISION MATRIX: Features vs. Use Cases

| Feature                  | Financial Services  | Healthcare     | Retail        |
|--------------------------|---------------------|----------------|---------------|
| [[00_Master_Links_and_Intro|IAM]]                      | High                | Medium         | Low           |
| [[Master/Srinivas_Notes/VPC|VPC]] [[appsync|Security]]             | Medium              | Medium         | High          |
| [[kms]] Encryption           | High                | High           | Medium        |
| [[00_Master_Links_and_Intro|CloudTrail]]               | Medium              | Medium         | Low           |

#### 2026 UPDATES:
- Enhanced [[00_Master_Links_and_Intro|IAM]] [[policies]] with more granular controls.
- Updated network [[appsync|security]] features in VPCs.

> [!warning] Quota
[[config|AWS Config]] recording limits (e.g., max 2,500 resource configurations recorded).

#### EXAM SCENARIOS:
1. **[[00_Master_Links_and_Intro|IAM]]**: Scenario where a user needs access to specific services but not [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]].
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|Security]]**: Setting up secure subnets and configuring NACLs for a multi-VPC environment.

#### INTERACTION PATTERNS:
- [[00_Master_Links_and_Intro|IAM]] [[policies]] interact with other AWS resources to enforce access controls.
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] components (subnets, [[appsync|security]] groups) interact to create a secure network architecture.

#### STRATEGIC ALIGNMENT:
1. **[[00_Master_Links_and_Intro|IAM]] [[policies]]**: Essential for securing services and controlling user access.
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|Security]]**: Fundamental for protecting network-based workloads.
3. **[[kms]] Encryption**: Critical for data at rest protection.

> [!check] Implementation
[[00_Master_Links_and_Intro|IAM]] [[policies]] interact with other AWS resources to enforce access controls.

#### PROFESSIONAL TONE & AUDIENCE:
This content is tailored to SAP-C02 candidates, focusing on the most relevant and high-weight topics as per recent exams.

#### PRIORITIZATION & CLARITY:
Focus on critical [[appsync|security]] features like [[00_Master_Links_and_Intro|IAM]], [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[appsync|Security]], and [[kms]] while providing clear explanations for complex concepts.

> [!danger] Distractor
Misunderstanding the number of [[appsync|security]] groups per instance can lead to misconfiguration and potential vulnerabilities.

#### GROUNDING & VALIDATION:
References official AWS documentation to ensure accuracy and alignment with the latest exam blueprint.

#### ARCHITECTURAL TRADE-OFFS:
- Trade-offs between complexity and [[appsync|security]] (e.g., more granular [[00_Master_Links_and_Intro|IAM]] [[policies]] vs. simpler access control).
- Cost considerations in encryption at rest vs. in transit.