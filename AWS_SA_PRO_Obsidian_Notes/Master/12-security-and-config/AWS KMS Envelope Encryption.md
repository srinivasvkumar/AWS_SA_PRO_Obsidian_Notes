```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[AWS KMS]] Envelope Encryption

#### Under-the-Hood Mechanics:
Envelope encryption is a technique used to encrypt data using two keys: a master key (managed by [[AWS KMS]]) and a data key. The process involves the following steps:

1. **Data Key Generation**: A symmetric data key is generated and used to encrypt the plaintext data.
2. **Encryption with Data Key**: The plaintext data is encrypted with the data key, producing ciphertext.
3. **Encryption of Data Key**: The data key itself is then encrypted using a master key managed by [[AWS KMS]].
4. **Secure Storage**: Both the encrypted data and the encrypted data key are stored together.

**Consistency Models:**
- The encryption process uses synchronous calls to ensure atomicity between generating, encrypting with, and storing both keys.
- Data consistency in storage ensures that if a failure occurs during any of these steps, the entire operation is rolled back or retried.

#### Exhaustive Feature List:
1. **Symmetric Encryption**: Utilizes [[AES-GCM]] for strong encryption standards.
2. **Key Management**: [[AWS KMS]] manages master keys (CMKs) to ensure security and compliance.
3. **Data Key Generation**: Can generate data keys with different lengths (e.g., 128-bit or 256-bit).
4. **Multi-Region Support**: Master keys can be used across multiple regions, ensuring global consistency.
5. **Key Rotation**: Automatic rotation of master keys to enhance security.
6. **Audit Logging**: Integration with [[AWS CloudTrail]] for logging all KMS operations.
7. **Access Control**: [[IAM]] policies and key policies define who can use which KMS keys.

#### Step-by-Step Implementation:
1. **Create a CMK in AWS KMS**:
   ```sh
   [[00_Master_Links_and_Intro|aws kms]] create-key --description "My Master Key" --policy file://key-policy.json
   ```
2. **Generate Data Key**:
   ```sh
   [[00_Master_Links_and_Intro|aws kms]] generate-data-key --key-id <CMK_ID> --key-spec AES_256
   ```
3. **Encrypt Plaintext Data with Data Key**: Use the data key to encrypt your plaintext.
4. **Encrypt Data Key with CMK**:
   ```sh
   [[00_Master_Links_and_Intro|aws kms]] encrypt --ciphertext-blob fileb://data-key.bin --key-id <CMK_ID>
   ```
5. **Store Encrypted Data and Encrypted Data Key**: Ensure both are stored securely together.

#### Technical Limits (2026):
- Maximum key length: 256 bits.
- Number of active keys per region: Soft limit of 1,000; contact [[AWS Support]] for higher limits.
- Frequency of key rotation: At least once every year; recommended more frequently for high-security environments.

#### CLI & API Grounding:
- **CLI Commands**:
  - `aws kms create-key`
  - `aws kms generate-data-key`
  - `aws kms encrypt`
  - `aws kms decrypt`
- **API Flags**: Refer to [[AWS KMS]] API documentation for detailed flags and parameters.

#### Pricing & TCO:
- Pricing based on the number of cryptographic operations (encryption/decryption, key generation).
- Cost optimization strategies include using multi-regional keys and minimizing unnecessary operations.
- Consider using AWS Budgets to monitor and control costs associated with [[AWS KMS]] usage.

#### Competitor Comparison:
- **Azure Key Vault**: Similar functionality but lacks some advanced features like automatic rotation out-of-the-box.
- **Google Cloud KMS**: Comparable in feature set but integration with other Google services may offer advantages depending on existing infrastructure.

#### Integration:
- **CloudWatch**: Integration for monitoring and logging all encryption/decryption operations.
- **IAM**: Fine-grained access control via IAM policies to manage who can use which keys.
- **VPC Endpoints**: Enable VPC endpoints for secure network communication between AWS services and KMS.

#### Use Cases:
1. **Enterprise Data Protection**: Encrypt sensitive data at rest in [[Amazon S3]] or [[rds]] using envelope encryption.
2. **Compliance Requirements**: Meet regulatory requirements by ensuring data is encrypted with strong keys managed externally via KMS.

#### Diagrams:
- Placeholder for diagram: [Envelope Encryption Architecture]
  - Trade-offs: Increased complexity vs enhanced security.

#### Exam Traps:
1. Confusing between symmetric and asymmetric encryption in key generation.
2. Misunderstanding the role of CloudWatch in monitoring [[AWS KMS]] operations.

#### Decision Matrix:
| Features               | Use Cases                             |
|------------------------|---------------------------------------|
| Symmetric Encryption   | Protecting sensitive data in S3 or RDS|
| Key Rotation           | Ensuring compliance with security policies|
| Multi-Region Support   | Global deployment consistency         |

### Audit of AWS KMS Envelope Encryption

#### Enhancement Requirements:

1. **Distractor Analysis**:
   - **Scenario 1**: **Use Case: Data in Transit Protection**
     > [!danger] Distractor
     While AWS KMS can be used to encrypt and decrypt data keys, it is not designed for the encryption of data in transit. Instead, services like [[AWS Certificate Manager (ACM)]] or SSL/TLS protocols should be utilized.
   - **Scenario 2**: **Use Case: Encrypting Data at Rest on EC2 Instances**
     > [!danger] Distractor
     For encrypting data at rest directly on an Amazon EC2 instance, using EBS volume encryption would be more appropriate. KMS can provide the keys used by [[ebs]] for encryption but does not directly encrypt the data itself.
   - **Scenario 3**: **Use Case: Data Masking**
     > [!danger] Distractor
     AWS KMS is designed for key management and encryption/decryption operations, not for data masking or anonymization purposes. Services like [[AWS DMS]] (Data Migration Service) with the Data Masking feature would be more appropriate.
   - **Scenario 4**: **Use Case: Server-Side Encryption of S3 Objects**
     > [!danger] Distractor
     While KMS can provide encryption keys, S3 server-side encryption is managed by S3 itself using [[SSE-KMS]]. AWS KMS alone does not handle the actual object-level encryption but provides the key material.

2. **The "Most Correct" Logic**:
   - **Performance vs. Cost Trade-offs**:
     > [!warning] Quota
     Performance: Envelope encryption helps in achieving better performance for encrypting large amounts of data by reducing the number of cryptographic operations on sensitive keys.
     > [!warning] Quota
     Cost: While KMS itself is relatively inexpensive (per-GB costs for envelope encryption), frequent key rotation and complex key management can increase operational overhead, thereby impacting cost.

3. **Enterprise Governance**:
   - **AWS Organizations**: Use organizational units to categorize different departments or projects to manage resources more efficiently.
   - **Service Control Policies (SCPs)**: Implement SCPs to restrict access to KMS keys based on least privilege principles, ensuring only authorized services and users can manage encryption operations.
   - **Resource Access Manager (RAM)**: Share KMS keys across accounts within an organization while maintaining security and compliance.
   > [!abstract] Exam Tip
   Use [[cloudtrail]] for auditing all actions taken with AWS KMS, including tracking key usage, creation, and deletion.

4. **Tier-3 Troubleshooting**:
   - **Complex Failure Modes**:
     > [!danger] Distractor
     *KMS Key Policy Conflicts*: In multi-account scenarios, conflicting IAM roles or cross-account policies can lead to unauthorized access attempts.
     > [!danger] Distractor
     *Rotation Issues*: Misconfigured rotation schedules might lead to key availability issues, disrupting encryption/decryption processes.
     > [!warning] Quota
     *Throttling Limits*: Exceeding the request limits for KMS operations can cause throttling errors, impacting system performance.

5. **Decision Matrix**:
| Use Case                      | Solution                        |
|-------------------------------|---------------------------------|
| Encrypt data at rest in EBS   | AWS KMS + EBS Encryption        |
| Protect sensitive API keys    | [[AWS Secrets Manager]]         |
| Securely manage encryption keys for S3 objects | SSE-KMS (Server-Side Encryption with AWS KMS) |
| Encrypt large volumes of data efficiently | Envelope Encryption using AWS KMS |

6. **Recent Updates**:
   - **GA Features 2026**: 
     > [!check] Implementation
     *Automatic Key Rotation*: Improved key rotation policies and automation.
     > [!check] Implementation
     *Enhanced Key Policies*: More granular control over who can use keys and under what conditions.
   - **Deprecated Items**:
     > [!danger] Distractor
     *Legacy KMS APIs*: Certain older API calls will be deprecated in favor of newer, more secure methods.

By incorporating these enhancements, the draft for AWS KMS Envelope Encryption is enriched with detailed insights into its correct application, performance considerations, governance strategies, troubleshooting tips, and recent updates.
```