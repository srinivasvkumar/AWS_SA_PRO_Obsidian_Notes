```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### AWS Shield Advanced Deep Dive

#### Overview of [[AWS Shield Advanced]]
[[AWS Shield Advanced]] is a managed Distributed Denial of Service (DDoS) protection service that safeguards applications running on the Amazon Web Services platform from DDoS attacks. It provides comprehensive, always-on monitoring and automatic inline mitigation to ensure availability of your resources.

---

### 1. **Distractor Analysis: Scenarios Where [[shield|AWS Shield]] Advanced is a "Wrong" Answer**
> [!danger] Distractor
- **Scenario 1:** You need to protect an application that runs outside the [[AWS]] environment (e.g., on-premises or another cloud provider). 
    - *Reason:* [[AWS Shield Advanced]] only protects resources within the AWS ecosystem.
  
- **Scenario 2:** Your primary concern is data encryption at rest and in transit, rather than DDoS protection.
    - *Reason:* While [[AWS Shield Advanced]] offers protection against DDoS attacks, it does not address other [[appsync|security]] concerns like encryption or access control.

- **Scenario 3:** You need to protect a server that has no public IP address (e.g., an [[00_Master_Links_and_Intro|EC2]] instance in a private subnet).
    - *Reason:* For resources without public IPs, the benefit of [[AWS Shield Advanced]] is limited as DDoS attacks generally target publicly accessible services.

---

### 2. **The "Most Correct" Logic: Trade-offs Between Performance and Cost**
> [!warning] Quota
- **Performance vs. Cost:**
    - *Performance:* [[AWS Shield Advanced]] provides high-performance DDoS mitigation with near-zero latency, ensuring your applications remain available during an attack.
    - *Cost:* While [[shield|AWS Shield]] Standard is free for all AWS customers, the advanced version requires a monthly fee per protected IP address. The cost-effectiveness of this solution depends on the size and complexity of your infrastructure and your exposure to DDoS risks.

---

### 3. **Enterprise Governance: Layers for [[AWS Organizations]], SCPs, [[ram]], and [[00_Master_Links_and_Intro|CloudTrail]]**
- **[[AWS Organizations]]:** Utilize [[organizations|AWS Organizations]] to centrally manage multiple accounts within an organization. This helps in applying consistent [[appsync|security]] [[policies]] across all accounts.
    - *Example:* Use organizational units (OUs) to group related accounts and apply service control [[policies]] (SCPs) that enforce specific permissions for [[shield]] Advanced usage.

- **Service Control [[policies]] ([[SCP]]):** SCPs can be used to restrict or allow access to [[AWS Shield Advanced]] features, ensuring compliance with your organization’s [[appsync|security]] [[policies]].
    - *Example:* You could configure an [[SCP]] that denies the creation of [[shield]] Advanced resources unless a specific tag is applied.

- **Resource Access Manager ([[ram]])**: [[ram]] enables you to share [[shield]] Advanced protections across different accounts within your organization, allowing centralized management and cost-sharing.
    - *Example:* Share a resource protection group with multiple AWS accounts using [[ram]], ensuring consistent DDoS protection for shared resources.

- **[[AWS CloudTrail]]:** To monitor and log access to [[Shield Advanced]] services, enable [[00_Master_Links_and_Intro|CloudTrail]]. This helps in auditing who accessed the service, what actions they took, and when.
    - *Example:* Configure [[00_Master_Links_and_Intro|CloudTrail]] to capture API calls made to [[shield|AWS Shield]] Advanced, such as `EnableProactiveEngagement` or `CreateProtection`.

---

### 4. **Tier-3 Troubleshooting: Complex Failure Modes**
> [!warning] Quota
- **[[kms]] Key Policy Conflicts:** When configuring [[Shield Advanced]] with [[00_Master_Links_and_Intro|AWS KMS]] (Key Management Service) for encryption keys used in [[vpc-flow-logs|logging]] and auditing, ensure that the key [[policies]] are correctly set up to allow necessary access.
    - *Failure Mode:* If there’s a conflict or incorrect policy setup, it could lead to [[shield]] Advanced not being able to encrypt logs properly, leading to data exposure risks.
    - *Troubleshooting Steps:*
        1. Verify [[kms]] permissions for the [[00_Master_Links_and_Intro|IAM]] roles used by [[Shield Advanced]].
        2. Check if the key policy and CMK settings allow necessary actions (e.g., `kms:Encrypt`, `kms:Decrypt`).
        3. Ensure that the CMK ARN is correctly configured in the [[shield]] Advanced setup.

---

### 5. **Decision Matrix: Use Case vs. Solution Table**

| **Use Case**                           | **Solution**                         |
|----------------------------------------|--------------------------------------|
| Protect a public-facing web application from DDoS attacks | [[AWS Shield Advanced]]                  |
| Ensure availability for global audiences with minimal latency | Global distribution of protected resources via [[cloudfront]] and [[Route 53]] with [[shield]] Advanced |
| Secure APIs running in a [[Master/Srinivas_Notes/VPC|VPC]] without public IPs | Use Network Load Balancer ([[NLB]]) as the entry point and enable [[shield]] Advanced on [[NLB]] |
| Monitor DDoS attack trends and incident responses | Enable [[shield]] Advanced's detailed analytics and [[vpc-flow-logs|logging]] features, integrate with [[AWS CloudWatch]] for monitoring |
| Centralize DDoS protection management across multiple accounts | Utilize [[organizations|AWS Organizations]] and SCPs to apply consistent [[policies]] |

---

### 6. **Recent Updates: Flag GA Features and Deprecated Items for 2026**
- **GA Features (as of 2026):** 
    - Enhanced integration with [[AWS CloudFormation]] for automated deployment.
    - Improved analytics dashboard for more granular attack visibility.

- **Deprecated Items:** 
    - Legacy API endpoints that were deprecated in favor of newer, more secure APIs.
    - Specific [[vpc-flow-logs|logging]] formats that are no longer supported due to updates in compliance standards.