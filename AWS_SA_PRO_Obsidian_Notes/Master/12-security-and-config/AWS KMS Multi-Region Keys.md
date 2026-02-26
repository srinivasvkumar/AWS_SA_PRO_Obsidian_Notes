```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### UNDER-THE-HOOD MECHANICS:
[[AWS Key Management Service (KMS)]] Multi-Region Keys ([[MRAKs]]) enable you to create and use encryption keys across multiple regions without needing to replicate them. Under the hood, MRAKs are designed to provide a consistent experience by allowing key operations in any [[AWS region|regions]] where they're enabled.

1. **Data Flow**: When a request is made to use an MRAK for encryption or decryption, the operation is routed to the primary region associated with that key. The primary region handles all cryptographic operations and then returns the result back to the client.
2. **Consistency Models**: [[AWS KMS]] uses strong consistency models to ensure that updates to key material are immediately visible across all enabled regions.
3. **Underlying Infrastructure Logic**: MRAKs maintain a single, authoritative copy of the key in the primary region and synchronize this state with other regions as needed. This ensures that any changes made to the key (e.g., enabling/disabling) propagate globally.

### EXHAUSTIVE FEATURE LIST:
1. **Cross-Region Operations**: Operate on keys from different [[AWS regions]].
2. **Key State Synchronization**: Automatic synchronization of key [[step-functions|states]] across enabled regions.
3. **Primary Region Management**: Specify a primary region that handles all cryptographic operations.
4. **Region Enablement/Disablement**: Control the regions where an MRAK is available using the console or API.
5. **Automatic Propagation**: Changes in key state (enabled/disabled) are automatically propagated to all enabled regions.

### STEP-BY-STEP IMPLEMENTATION:
1. **Create a Multi-Region Key**:
   - Navigate to [[AWS KMS Console]].
   - Select "Create key" and choose the "Multi-Region key" option.
2. **Specify Primary Region**: Choose the primary region where the key will reside.
3. **Enable Regions**: Specify which regions should have access to this key.
4. **Configure Key Policies/Permissions**: Define [[IAM policies]] and permissions for accessing the key.

### TECHNICAL LIMITS (2026):
- **Soft Quotas**:
  - Maximum number of enabled regions: 15 per MRAK.
  - Maximum number of aliases per region: 1,000.
- **Hard Quotas**:
  - Each AWS account can have up to 2,000 customer-managed keys (including MRAKs).

### CLI & API GROUNDING:
- **AWS CLI Commands**: 
  ```sh
  aws kms create-multi-region-key --bypass-policy-lockout-safety-check
  aws kms enable-key-rotation --key-id <key_id>
  ```
- **API Flags**:
  - `CreateMultiRegionKey`: Creates a new multi-region key.
  - `EnableKeyRotation`: Enables automatic rotation of the key.

### PRICING & TCO:
- **Cost Drivers**: Pay per request (create, encrypt, decrypt) and storage for each enabled region.
- **Optimization Strategies**:
  - Enable MRAKs only in regions where they are needed to avoid unnecessary costs.
  - Use [[IAM policies]] to limit access and reduce the number of requests.

### COMPETITOR COMPARISON:
- **Azure Key Vault**: Offers a similar feature called "Geographically Replicated Keys", but with different region enablement controls.
- **Google Cloud [[kms]]**: Supports cross-region keys, but requires manual configuration for each region.

### INTEGRATION:
- **[[cloudwatch]]**:
  - Metrics and logs can be sent to [[cloudwatch]] for monitoring key usage.
  - Example: `aws kms generate-data-key --key-id <key_id>` generates a data key with metrics available in [[cloudwatch]].
- **[[00_Master_Links_and_Intro|IAM]]**: Use [[00_Master_Links_and_Intro|IAM]] [[policies]] to control access to MRAKs across regions.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: No direct integration, but [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]] endpoints can be used for secure communication.

### USE CASES:
1. **Global Data Encryption**: Encrypting data stored in different regions using a consistent key.
2. **Multi-Region Workloads**: Ensuring that applications deployed across multiple AWS regions have access to the same encryption keys.
3. **Compliance Requirements**: Meeting regulatory requirements by maintaining strict control over key usage and state synchronization.

### DIAGRAMS:
1. **MRAK Architecture**:
   - Placeholder for a diagram showing primary region handling cryptographic operations and other regions accessing it.
2. **Data Flow Diagram**:
   - Placeholder for a diagram illustrating the flow of data between regions and the primary region.

> [!abstract] Exam Tip
Misconception: Thinking that MRAKs automatically replicate key material across regions (they do not; they only synchronize state).

> [!warning] Quota
Each AWS account can have up to 2,000 customer-managed keys (including MRAKs).

### DECISION MATRIX:
| Feature                        | Use Case                       |
|--------------------------------|-------------------------------|
| Cross-Region Operations        | Global data encryption         |
| Key State Synchronization      | Multi-region application deployments     |
| Region Enablement/Disablement  | Compliance and access control  |

> [!check] Implementation
Ensure all quotas, features, and CLI/API references are up-to-date with the latest [[00_Master_Links_and_Intro|AWS KMS]] documentation.

### EXAM SCENARIOS:
- **Scenario**: Determining which regions to enable for an MRAK based on application deployment.
- **Scenario**: Configuring [[00_Master_Links_and_Intro|IAM]] [[policies]] for cross-region key access.

> [!danger] Distractor
If the requirement is to replicate keys across regions within a single account, but not across different accounts or regions, [[AWS Key Management Service (KMS)]] Custom Key Stores might be considered instead of Multi-Region Keys.