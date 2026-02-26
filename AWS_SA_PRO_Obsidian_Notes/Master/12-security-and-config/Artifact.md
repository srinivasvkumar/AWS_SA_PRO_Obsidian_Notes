## Advanced Architecture
At its core, AWS Artifact is a unified repository of [[appsync|security]] and compliance reports, as well as assurance programs, that allows users to demonstrate their organization's [[appsync|security]] posture and compliance with various industry standards and [[iam|best practices]]. It operates at a global scale, offering centralized management of findings and evidence across multiple accounts and regions.

Internally, Artifact uses a combination of AWS services like Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS KMS]], [[lambda|AWS Lambda]], and [[organizations|AWS Organizations]] to store, encrypt, process, and distribute reports. The service employs fine-grained access control using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], ensuring that only authorized users can access specific report types or sections. Additionally, Artifact supports both [[sns]] and [[cloudwatch]] Events for notifications, enabling automation and integration with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]].

## Comparison & Anti-Patterns
While AWS Artifact provides valuable compliance and [[appsync|security]] documentation, it may not be suitable in certain scenarios due to [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] such as regional availability or lack of API support. In these cases, consider utilizing alternative services or tools:

| Service                    | Use Cases                                                         |
| --------------------------- | --------------------------------------------------------------- |
| [[security-hub|AWS Security Hub]]           | Centralized [[appsync|security]] alerts and findings aggregation            |
| [[config|AWS Config]]                 | Compliance checks and custom rules for resource configurations      |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Firewall Manager]]        | Managing AWS WAF and Network Firewall across accounts             |
| [[shield|AWS Shield]]                | DDoS protection and mitigation                                   |
| 3rd-party scanning tools   | Periodic vulnerability assessments and penetration testing     |

Anti-patterns include treating Artifact as a standalone [[appsync|security]] solution rather than an integrated component within a larger [[appsync|security]] strategy, assuming Artifact covers all [[appsync|security]] aspects (it doesn't), and relying solely on provided reports without further investigation or remediation.

## [[appsync|Security]] & Governance
To implement advanced [[appsync|security]] and governance features in AWS Artifact, consider the following techniques:

### Fine-Grained Access Control
Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to restrict access to specific report types or sections based on user roles. For example, limit the "AWS Foundational [[appsync|Security]] [[iam|Best Practices]]" report access to senior [[appsync|security]] engineers while granting general read access to all employees.

Example JSON policy snippet:
```json
{
  "Effect": "Allow",
  "Action": [
    "artifact:Describe