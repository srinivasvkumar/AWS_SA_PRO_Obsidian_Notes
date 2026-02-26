**Advanced Architecture**

[[CloudHSM]] is a cloud-based hardware [[appsync|security]] module (HSM) service that enables you to manage, deploy, and scale cryptographic operations. It offers FIPS 140-2 Level 3 validated HSMs. The internal architecture consists of four main components: Safety Capsule, Key Broker, HSM Nodes, and Administration Server.

*The Safety Capsule*: A hardware component that stores sensitive information like HSM credentials, root keys, and PINs.

*Key Broker*: Acts as an intermediary between HSM nodes and clients, managing key lifecycle events.

*HSM Nodes*: Perform cryptographic functions such as encryption, decryption, and [[kms|signing]].

*Administration Server*: Provides a web-based interface for [[CloudHSM]] management.

[[CloudHSM]] supports global scale by distributing HSM instances across multiple Availability Zones (AZs) within a region. This ensures high availability and redundancy. Data replication occurs at the partition level, enabling localized data recovery in case of failure.

**Comparison & Anti-Patterns**

| Service           | Use Case                                                              |
| ----------------- | -------------------------------------------------------------------- |
| [[CloudHSM]]         | Regulated industries requiring FIPS 140-2 Level 3 HSM                |
| [[kms]]              | Managed symmetric and asymmetric keys, no direct user control       |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]]   | Storing, retrieving, and rotating application secrets                |
| Hardware Device   | Local, physical device providing HSM functionality                    |

Anti-pattern: Using [[CloudHSM]] when [[kms]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] would suffice due to lower complexity and cost.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON statements allowing fine-grained access control. Here's an example granting `cloudhsm:DescribeHsm` permission:
```json
{
    "Effect": "Allow",
    "Action": [
        "cloudhsm:DescribeHsm"
    ],
    "Resource": [
        "*"
    ]
}
```
Cross-account access requires creating a trust relationship through an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role. An Organization Service Control Policy ([[SCP]]) can enforce [[CloudHSM]] usage within member accounts.

**Performance & Reliability**

Throttling limits exist per account and reset every hour. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve distributing HSM instances across multiple AZs and regions.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include [[billing]] based on provisioned HSM capacity and active usage. Calculate costs with the [AWS Pricing Calculator](https://calculator.aws).

**Professional Exam Scenarios**

Scenario 1:
An organization needs to store credit card numbers securely while meeting PCI DSS requirements. They have existing AWS infrastructure and want to minimize operational overhead. Which solution meets these needs?

Correct Answer: [[CloudHSM]].
Explanation: [[CloudHSM]] provides FIPS 140-2 Level 3 HSMs required by PCI DSS. By offloading cryptographic operations to [[CloudHSM]], operational overhead is minimized.

Incorrect Answer: [[kms]].
Explanation: While [[kms]] is managed by AWS, it does not offer direct user control over cryptographic operations and lacks FIPS 140-2 validation.

Scenario 2:
A company wants to ensure their [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] follow [[iam|best practices]] for securing a multi-account setup with [[CloudHSM]]. What should they do?

Correct Answer: Implement an Organization [[SCP]] enforcing [[CloudHSM]] usage within member accounts.
Explanation: [[organizations]] can create an [[SCP]] that allows specific actions related to [[CloudHSM]]. This ensures consistent enforcement across all member accounts.

Incorrect Answer: Create a custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy for each account.
Explanation: Custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] increase complexity and maintenance burden. [[organizations]] benefit from centralized governance via SCPs.