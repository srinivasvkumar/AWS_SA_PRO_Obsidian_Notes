```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Amazon S3]] Object Lock and [[Vault Lock]]

#### UNDER-THE-HOOD MECHANICS:
[[Amazon S3 Object Lock]] is a feature that enables you to lock objects in your [[S3 buckets]] to prevent them from being deleted or modified for a specified period. This feature uses the WORM (Write Once Read Many) model, ensuring compliance with regulatory requirements.

- **Internal Data Flow**: When Object Lock is enabled on an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket, all PUT and DELETE requests go through additional checks to ensure they comply with retention [[policies]].
- **Consistency Models**: Object Lock guarantees strong consistency for all operations within the [[S3 bucket]]. This means that once a write operation is confirmed, subsequent read operations will always return the latest data.
- **Underlying Infrastructure Logic**: The feature leverages AWS's infrastructure to enforce retention rules at the object level. Each object has metadata attached that defines its [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]].

#### EXHAUSTIVE FEATURE LIST:
1. **Retention Periods** - [[Compliance]] (legal holds) and Governance modes.
2. **Legal Holds** - Enforce WORM on specific objects.
3. **Versioning**: Object Lock works in conjunction with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Versioning, enabling version locking.
4. **Audit Logs**: Detailed logs via [[AWS CloudTrail]] for all actions affecting locked objects.

#### STEP-BY-STEP IMPLEMENTATION:
1. Enable [[S3 Versioning]] and Bucket [[policies]] to enforce access control.
2. Create a bucket policy that requires Object Lock.
3. Set retention periods on the bucket either at the governance or compliance level.
4. Apply legal holds using [[AWS Management Console]], SDKs, or APIs.

#### TECHNICAL LIMITS (2026):
- **Soft Quotas**: Maximum of 1 million objects per [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket under a single account.
- **Hard Quotas**: None explicitly defined for Object Lock itself; [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] are inherited from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket limits.

#### CLI & API GROUNDING:
```sh
# Enable versioning on the bucket
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled

# Set retention policy
aws s3api put-object-lock-configuration \
    --bucket my-bucket \
    --object-lock-configuration "ObjectLockEnabled=Enabled,Rule={DefaultRetention={Mode=governance,Days=365}}"

# Apply legal hold
aws s3api put-object-retention \
    --bucket my-bucket \
    --key example.txt \
    --version-id <VERSION_ID> \
    --bypass-governance-retention \
    --retention "Mode=COMPLIANCE,RetainUntilDate=2026-12-31T23:59:59.000Z"
```

#### PRICING & TCO:
- **Cost Drivers**: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] standard storage costs plus additional charges for Object Lock operations.
- **Optimization Strategies**: Use lifecycle [[policies]] to transition objects between different [[S3 Storage Classes]] based on age.

#### COMPETITOR COMPARISON:
- **Azure Blob Storage with Immutable Storage** - Similar WORM capabilities but Azure lacks the comprehensive audit and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|legal hold]] features that [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] provides.
- **Google Cloud Storage Nearline/WARM [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|Storage Classes]]**: Offers retention [[policies]], but lacks fine-grained control over governance modes.

#### INTEGRATION:
- **[[cloudwatch]]**: Monitor [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket events for any changes to objects.
- **[[00_Master_Links_and_Intro|IAM]] [[policies]]**: Enforce granular access controls on buckets and objects with specific permissions required for Object Lock operations.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]]**: Securely connect to [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], minimizing exposure.

#### USE CASES:
1. **Regulatory Compliance**: Financial institutions must comply with regulations like GDPR or HIPAA.
2. **Archival Storage**: Long-term storage of documents and media that cannot be modified once created.

#### DIAGRAMS:
- Placeholder for architectural diagram showing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket setup with Object Lock enabled, integrating [[00_Master_Links_and_Intro|IAM]] roles and [[cloudwatch]] monitoring.

> [!abstract] Exam Tip
1. Misunderstanding the difference between Governance Mode and Compliance Mode.
2. Forgetting to enable versioning before applying Object Lock.
3. Ignoring the importance of lifecycle [[policies]] in cost management.

#### DECISION MATRIX:
| Feature               | Use Case Example                          |
|-----------------------|-------------------------------------------|
| Retention Periods     | Financial Records (Regulatory compliance) |
| Legal Holds           | Legal Discovery Processes                 |
| [[Master/Srinivas_Notes/S3|S3]] Versioning         | Archiving Historical Data                 |

> [!check] Implementation
1. Scenario: You need to ensure that certain financial documents are immutable for five years due to compliance requirements.
   - Solution: Use [[S3 Object Lock]] with a Governance Mode [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] of 5 years.

2. Scenario: A client needs to implement legal holds on specific documents during an ongoing audit.
   - Solution: Apply legal holds using the Compliance Mode in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]].

#### INTERACTION PATTERNS:
- **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Bucket Policy Integration**: [[policies]] enforce compliance with Object Lock rules.
- **[[00_Master_Links_and_Intro|CloudTrail]] and [[cloudwatch]]**: Monitor and log all actions affecting locked objects.

> [!warning] Quota
Technical Limits (2026):
- Soft Quotas: Maximum of 1 million objects per [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket under a single account.
- Hard Quotas: None explicitly defined for Object Lock itself; [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] are inherited from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket limits.

#### EXAM SCENARIOS:
1. Scenario: You need to ensure that certain financial documents are immutable for five years due to compliance requirements.
   - Solution: Use [[S3 Object Lock]] with a Governance Mode [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]] of 5 years.

2. Scenario: A client needs to implement legal holds on specific documents during an ongoing audit.
   - Solution: Apply legal holds using the Compliance Mode in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]].

#### PROFESSIONAL TONE & AUDIENCE FOCUS:
Tailored specifically for SAP-C02 candidates, ensuring clarity and conciseness in delivering high-value technical breakdowns relevant to the exam blueprint.

> [!danger] Distractor
Below are several scenarios where Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] or Vault Lock would be an incorrect answer:

- **Scenario 1: Temporary Data Storage**: If the requirement is to store data temporarily and frequently update/delete objects, using Object Lock could be counterproductive as it restricts modifications.

- **Scenario 2: High-Frequency Transactions**: For applications requiring high-frequency read/write operations, such as real-time analytics or transactional databases, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] might introduce unnecessary overhead.

- **Scenario 3: Short-Term Compliance Requirements**: If the compliance requirements are only for a short period (e.g., less than 90 days), and the organization needs flexibility in managing the data lifecycle, [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|S3 Object Lock]] may be overkill.

- **Scenario 4: Data Archival with Frequent Access Needs**: For archival purposes where frequent access to the archived data is required, Vault Lock might not be ideal as it is designed for long-term immutability without frequent retrieval needs.

- **Scenario 5: Cost-Sensitive Use Cases**: If [[00_Master_Links_and_Intro|cost optimization]] is a primary concern and infrequent access to data is acceptable, using simpler [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] (e.g., [[00_Master_Links_and_Intro|Glacier]]) without Object Lock or Vault Lock could be more economical.

#### DECISION MATRIX:
| Use Case                  | Solution           |
|---------------------------|--------------------|
| Regulatory Compliance     | [[S3 Object Lock]] |
| Long-Term Data Archival   | [[Vault Lock]]         |
| Short-Term Immutability   | [[Master/Srinivas_Notes/S3|S3]] Versioning      |
| Frequent Data Modifications | Standard [[Master/Srinivas_Notes/S3|S3]]       |

#### 2026 UPDATES:
- Ensure adherence to the latest AWS documentation and whitepapers.
- Consider any upcoming updates to services or pricing models.

> [!warning] Quota
Technical Limits (2026):
- Soft Quotas: Maximum of 1 million objects per [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket under a single account.
- Hard Quotas: None explicitly defined for Object Lock itself; [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] are inherited from [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket limits.