```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### AWS Config Conformance Packs Deep-Dive

#### UNDER-THE-HOOD MECHANICS:
[[AWS Config Conformance Packs]] are used to assess the compliance of your [[AWS resources]] against predefined rules and [[policies]]. The underlying mechanics involve:

- **Data Flow**: [[config|Conformance Packs]] use [[AWS Config]]'s ability to collect, store, and query resource configuration changes.
- **Consistency Models**: Rules within a conformance pack trigger evaluations that can be configured for periodic or on-demand execution.
- **Underlying Infrastructure Logic**: [[config]] rules are evaluated against your resources, and the results are stored in [[config|AWS Config]]’s state database.

#### EXHAUSTIVE FEATURE LIST:
1. **Predefined Compliance Packs**: Out-of-the-box packs for various compliance standards (e.g., [[CIS]], [[PCI-DSS]]).
2. **Custom Rule Creation**: Ability to define custom rules using [[AWS Lambda]].
3. **Resource Types**: Support for a wide range of resource types including [[EC2 instances]], [[S3 buckets]], and [[IAM roles]].
4. **Multi-Account Management**: Use [[AWS Organizations]] to manage [[config|conformance packs]] across multiple accounts.
5. **Remediation Actions**: Automatically apply remediations based on evaluation results.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set Up [[config]] Recorder**: Enable the [[config]] recorder in each account or via an organization.
2. **Create a Conformance Pack**:
   - Use [[AWS Management Console]], CLI, or SDKs.
   - Define rules and specify compliance packs.
3. **Deploy Across Accounts** (if using [[organizations]]):
   - Tag accounts as needed for deployment.
4. **Monitor Compliance**: View results in the [[config]] console.

#### TECHNICAL LIMITS (2026):
- **Soft Quotas**: 10 [[config|conformance packs]] per region, can be increased via support request.
- **Hard Quotas**: 50 managed rules and 35 Lambda-backed rules per pack.

#### CLI & API GROUNDING:
- `aws configservice put-conformance-pack` for creating a new conformance pack.
- Use `--template-body` to specify the template with compliance rules.
- Example: 
```bash
aws configservice put-conformance-pack --conformance-pack-name MyConformancePack \
    --template-body file://my-config-template.yaml \
    --delivery-s3-bucket my-bucket \
    --delivery-s3-key-prefix conformancepacks/
```

#### PRICING & TCO:
- **Cost Drivers**: Cost depends on the number of resources being evaluated and how frequently evaluations are run.
- **Optimization Strategies**:
  - Use [[AWS Config]] pricing calculator to estimate costs.
  - Limit rule scope to relevant resource types and reduce evaluation frequency if possible.

#### COMPETITOR COMPARISON:
- **Azure Policy**: Similar in functionality but Azure-specific rules and compliance packs. Integrates tightly with [[Azure Monitor]] for insights.
- **Google Cloud [[appsync|Security]] Command Center**: Offers built-in [[policies]] but less granular control compared to AWS Config Conformance Packs.
- **Architectural Trade-offs**:
  - [[config|AWS Config]] is highly customizable, but requires more setup effort compared to out-of-the-box solutions like Google’s SCC.

#### INTEGRATION:
- **[[cloudwatch]]**: Notifications can be sent via [[sns]] for non-compliant resources.
- **[[00_Master_Links_and_Intro|IAM]]**: Roles and permissions are necessary for creating and managing [[config|conformance packs]].
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: [[config|Conformance Packs]] work across all regions but require proper network setup to access resources in private subnets.

#### USE CASES:
1. **PCI-DSS Compliance**: Deploy the PCI-DSS compliance pack to ensure adherence to payment card industry standards.
2. **CIS Benchmarks**: Use CIS benchmarks to secure [[EC2 instances]] and other critical services.
3. **Multi-Account Management**: Ensure consistent [[appsync|security]] [[policies]] across multiple AWS accounts in an organization.

#### DECISION MATRIX:
| Feature            | PCI-DSS Compliance | CIS Benchmarks | Multi-Account Management |
|--------------------|-------------------|----------------|-------------------------|
| Predefined Rules   | Yes               | Yes            | Yes                     |
| Custom Rule Support| Limited           | Extensive      | Yes                     |
| Remediation Actions| Basic             | Advanced       | Yes                     |

#### 2026 UPDATES:
- Ensure all quotas and features align with the latest AWS updates.
- Check for new compliance standards supported.

### Exam Tips
> [!abstract] Exam Tip
1. **Misconception**: [[config|Conformance Packs]] can be directly edited once created (they must be deleted and recreated).
2. **Soft vs. Hard Quotas**: Understanding the difference between soft and hard quotas for [[config|conformance packs]].
3. **Rule Evaluation Frequency**: Clarify that rules can evaluate resources on-demand or at a specified interval.

### Implementation Steps
> [!check] Implementation
1. Set up [[config]] Recorder.
2. Create a Conformance Pack via AWS Management Console, CLI, or SDKs.
3. Deploy across accounts using [[organizations]] if needed.
4. Monitor compliance in the [[config]] console.

### Quotas & [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]
> [!warning] Quota
- Soft Quotas: 10 [[config|conformance packs]] per region (can be increased).
- Hard Quotas: 50 managed rules and 35 Lambda-backed rules per pack.

### Distractors
> [!danger] Distractor
1. **Scenario**: Monitoring real-time changes is better handled by [[cloudtrail]] or [[eventbridge]].
2. **Scenario**: Fine-grained access control should be implemented with [[00_Master_Links_and_Intro|IAM]] roles and [[policies]], not [[config|Conformance Packs]] alone.
3. **Scenario**: Continuous [[appsync|security]] assessments are better managed using [[AWS Security Hub]].
4. **Scenario**: Data loss prevention (DLP) [[policies]] are best managed via [[AWS Macie]].
5. **Scenario**: Organizational-level compliance rules can be enforced more broadly with SCPs in [[organizations|AWS Organizations]].

#### Enterprise Governance
- **[[organizations|AWS Organizations]]**: Centralized management of compliance rules across multiple accounts using SCPs.
- **Service Control [[policies]] (SCPs)**: Restrict actions performed by [[00_Master_Links_and_Intro|IAM]] users and roles, ensuring only authorized changes to [[config|conformance packs]].
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Manage resource sharing within an organization to ensure consistent [[config|AWS Config]] settings.
- **[[00_Master_Links_and_Intro|CloudTrail]]**: Log API calls related to the creation, modification, or deletion of [[config|conformance packs]] for detailed audit trails.

#### Troubleshooting
**[[kms]] Key Policy Conflicts**
1. Verify that the [[00_Master_Links_and_Intro|IAM]] role used by [[AWS Config]] has necessary permissions to decrypt KMS-encrypted data.
2. Check [[kms]] key [[policies]] for any restrictive settings that might prevent access.
3. Ensure correct key aliases are referenced in configurations.

#### Decision Matrix

| Use Case                          | Solution                            |
|-----------------------------------|-------------------------------------|
| Monitoring real-time changes      | [[AWS CloudTrail]] / [[eventbridge]]    |
| Fine-grained access control       | [[00_Master_Links_and_Intro|IAM]] roles and [[policies]]              |
| Continuous [[appsync|security]] assessments   | [[AWS Security Hub]]                |
| Implementing DLP [[policies]]         | [[AWS Macie]]                       |
| Organizational compliance rules   | SCPs in [[organizations|AWS Organizations]]           |

#### Recent Updates
- **GA Features** (as of 2026):
  - Enhanced support for new resource types and additional rule configurations.
  
- **Deprecated Items** (as of 2026):
  - Older versions of predefined [[config|conformance packs]] that no longer align with current compliance standards.