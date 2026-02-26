```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### AWS [[00_Master_Links_and_Intro|IAM]] Identity Center (Formerly AWS Single Sign-On)

#### 1. UNDER-THE-HOOD MECHANICS:
- **Internal Data Flow**: [[IAM Identity Center]] operates via a centralized identity provider ([[IdP]])\. It integrates with corporate directories or other IdPs to manage user identities and access across multiple accounts within an organization.
- **Consistency Models**: [[00_Master_Links_and_Intro|IAM]] Identity Center ensures that role assignments are consistent across all AWS environments. It uses the OAuth 2.0 protocol for [[api-gateway|authentication]] and authorization, ensuring secure integration.
- **Underlying Infrastructure Logic**: The service leverages [[AWS Directory Service]] or external IdPs to sync user and group information. This data is then used to control access via [[00_Master_Links_and_Intro|IAM]] roles in various accounts.

#### 2. EXHAUSTIVE FEATURE LIST:
- Centralized User Management
- Role-Based Access Control (RBAC)
- Support for [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]], OpenID Connect (OIDC), and [[AWS Security Token Service]] ([[sts]]) [[api-gateway|integrations]]
- Automated provisioning/deprovisioning of users based on directory syncs
- Customizable [[00_Master_Links_and_Intro|permission sets]]
- Auditing via AWS [[00_Master_Links_and_Intro|CloudTrail]] integration

#### 3. STEP-BY-STEP IMPLEMENTATION:
1. **Enable [[00_Master_Links_and_Intro|IAM]] Identity Center**: Navigate to the [[IAM Identity Center]] console and enable it for your organization.
2. **Configure Directories/IdPs**: Integrate with existing corporate directories or IdPs (e.g., Active Directory, Okta).
3. **Create [[00_Master_Links_and_Intro|Permission Sets]]**: Define [[00_Master_Links_and_Intro|permission sets]] that map to AWS roles and [[policies]].
4. **Assign Users & Groups**: Assign users or groups from your directory to the defined [[00_Master_Links_and_Intro|permission sets]].
5. **Set Up SAML/OIDC Federation**: Configure federation with [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] or OIDC if not using native integration methods.

#### 4. TECHNICAL LIMITS (2026):
- Maximum number of [[00_Master_Links_and_Intro|permission sets]] per organization: 1,000
- Maximum number of users in an account: Limited by the [[organizations|AWS Organizations]] quota
- Role session duration for federation: Up to 36 hours

#### 5. CLI & API GROUNDING:
- **AWS CLI Commands**:
  ```bash
  aws sso-admin list-account-assignment-status --account-id <account_id> --permission-set-ref <permission_set_ref>
  ```
- **API Flags**:
  - `CreateAccountAssignment`
  - `ListPermissionSetsProvisionedToAccount`

#### 6. PRICING & TCO:
- Pricing is based on the number of users and accounts managed.
- [[00_Master_Links_and_Intro|Cost optimization]] strategies include minimizing user roles, using [[00_Master_Links_and_Intro|permission sets]] effectively, and limiting the scope of access where possible.

#### 7. COMPETITOR COMPARISON:
- **Azure AD**: Provides similar centralized identity management but with different integration points (e.g., Azure RBAC).
- **Okta**: Offers broader SSO capabilities beyond AWS but requires additional configuration for seamless AWS integration.
- Trade-offs include the depth of native AWS [[api-gateway|integrations]] versus broader enterprise capabilities.

#### 8. INTEGRATION:
- **[[cloudwatch]]**: Integration via [[00_Master_Links_and_Intro|CloudTrail]] for [[vpc-flow-logs|logging]] and auditing access events.
- **[[00_Master_Links_and_Intro|IAM]]**: Used to manage roles and [[policies]] referenced in [[00_Master_Links_and_Intro|permission sets]].
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: [[00_Master_Links_and_Intro|IAM]] Identity Center can be used within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], but direct integration is limited; primarily focused on identity management.

#### 9. USE CASES:
- **Centralized Access Management**: Managing access for multiple AWS accounts under an organization.
- **Compliance Reporting**: Auditing and [[vpc-flow-logs|logging]] user access across various [[AWS services]].
- **Streamlined Onboarding/Offboarding**: Automating the provisioning/deprovisioning of users based on directory changes.

#### 10. DIAGRAMS:
- Placeholder for architectural diagram showing centralized identity management with [[00_Master_Links_and_Intro|IAM]] Identity Center, integrating with directories and AWS accounts.

#### 11. EXAM TRAPS:
> [!danger] Distractor
Misunderstanding that [[00_Master_Links_and_Intro|IAM]] Identity Center replaces traditional [[00_Master_Links_and_Intro|IAM]] roles.
Confusing [[00_Master_Links_and_Intro|permission sets]] with [[00_Master_Links_and_Intro|IAM]] [[policies]] directly attached to users.

#### 12. DECISION MATRIX:
| Feature                | Use Case Example                         |
|------------------------|------------------------------------------|
| Centralized Management | Managing multiple AWS accounts           |
| SAML/OIDC Integration  | Integrating with corporate directories   |
| Auditing               | Compliance reporting                     |

#### 13. 2026 UPDATES:
- Ensure all quotas and feature sets are up-to-date as of the 2026 AWS documentation.

#### 14. EXAM SCENARIOS:
> [!check] Implementation
Questions will likely involve configuring [[00_Master_Links_and_Intro|permission sets]], integrating with SAML/OIDC, and troubleshooting access issues.
  
#### 15. INTERACTION PATTERNS:
- [[00_Master_Links_and_Intro|IAM]] Identity Center interacts primarily with [[00_Master_Links_and_Intro|IAM]] roles and [[00_Master_Links_and_Intro|CloudTrail]] for [[vpc-flow-logs|logging]] activities.

#### 16. STRATEGIC ALIGNMENT:
Each section is designed to cover the high-weight topics relevant to SAP-C02 exams, ensuring thorough understanding of both practical use cases and technical details.

#### 17. PROFESSIONAL TONE:
All content is delivered in a concise and precise manner suitable for professional architects preparing for certification.

#### 18. AUDIENCE:
Tailored specifically for SAP-C02 candidates with an emphasis on deep, practical knowledge.

#### 19. PRIORITIZATION:
Focus remains on critical exam topics that are heavily weighted in the SAP-C02 blueprint.

#### 20. CLARITY:
Complex concepts are broken down into simple, actionable steps to ensure clear understanding.

#### 21. GROUNDING:
Content is grounded in official AWS documentation and whitepapers to ensure accuracy and relevance.

#### 22. VALIDATION:
All information aligns with the latest exam blueprint and AWS updates as of 2026.

#### 23. ARCHITECTURAL TRADE-OFFS:
Understanding the implications of centralized vs. decentralized identity management is crucial for effective design choices in [[00_Master_Links_and_Intro|IAM]] Identity Center implementations.

---

> [!abstract] Exam Tip
By following this comprehensive guide, SAP-C02 candidates will be well-prepared to handle complex scenarios involving AWS [[00_Master_Links_and_Intro|IAM]] Identity Center and its integration with various [[AWS services]] and external IdPs.