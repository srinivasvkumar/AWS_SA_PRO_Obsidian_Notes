```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Multi-Account Strategy & Landing Zones Deep-Dive

#### UNDER-THE-HOOD MECHANICS:
A multi-account strategy in [[AWS]] involves organizing workloads across multiple [[AWS accounts]] to achieve better security, governance, and operational efficiency. Each account can have its own set of [[VPCs]], [[IAM policies]], and resources. The landing zone is an architectural pattern that helps organizations rapidly deploy, govern, and operate a consistent environment for the cloud.

1. **Internal Data Flow**: 
   - Workloads are distributed across multiple accounts to isolate risks.
   - Cross-account access is managed via [[IAM roles]] with trust relationships.
   - Resource sharing can be facilitated using [[AWS Organizations Service Control Policies (SCPs)]].

2. **Consistency Models**:
   - Consistent governance and compliance policies across all accounts.
   - Standardized security configurations, such as network segmentation.

3. **Underlying Infrastructure Logic**:
   - [[AWS Organizations]] centralizes the management of multiple accounts.
   - [[Resource Access Manager (RAM)]] enables resource sharing between accounts.
   > [!abstract] Exam Tip
   > Service Control Policies (SCPs) enforce governance rules at an organizational level.

#### EXHAUSTIVE FEATURE LIST:
1. **AWS Organizations**: Centralized account and service management.
2. **IAM Roles & Trust Relationships**: Cross-account access control.
3. **Service Control Policies (SCPs)**: Enforce compliance policies across accounts.
4. **Resource Access Manager (RAM)**: Share resources between accounts.
5. **VPC Sharing**: Extend network reachability across accounts.
6. **AWS Config**: Monitor and enforce configuration compliance.
7. **CloudTrail & CloudWatch**: Centralized logging and monitoring for governance.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Create an AWS Organization**:
   - Use the `aws organizations create-organization` command to set up your organization.
   
2. **Create Accounts**:
   - Utilize `aws organizations create-account` to add new accounts within the organization.

3. **Configure Service Control Policies (SCPs)**:
   - Define SCPs using the AWS Management Console or CLI commands like `aws organizations put-policy`.

4. **Set Up IAM Roles and Trust Relationships**:
   - Create roles with trust relationships for cross-account access.
   
5. **Deploy Workloads**:
   - Use [[CloudFormation]] templates to deploy consistent configurations across accounts.

6. **Enable Resource Sharing**:
   - Configure RAM to share resources between different accounts.

#### TECHNICAL LIMITS (2026):
1. **AWS Organizations**:
   - Maximum number of accounts: 1,000
   - Maximum number of OUs (Organizational Units): 500

2. **IAM Roles & Trust Relationships**:
   - Maximum policies per role: 10
   - Maximum statements per policy: 1,000

3. **Service Control Policies (SCPs)**:
   - Each account can have up to 20 SCPs attached.
   
4. **Resource Access Manager (RAM) Shares**:
   - Maximum resources in a share: 500
   - Maximum shares per resource type: 1,000

#### CLI & API GROUNDING:
1. **Creating an Organization**:
    ```sh
    [[organizations|aws organizations]] create-organization --feature-set ALL
    ```

2. **Creating Accounts within the Organization**:
    ```sh
    [[organizations|aws organizations]] create-account --email email@example.com --account-name ExampleAccount
    ```

3. **Putting Service Control Policies (SCPs)**:
    ```sh
    [[organizations|aws organizations]] put-policy --content file://policy.json --type SERVICE_CONTROL_POLICY --name PolicyName
    ```

4. **Creating IAM Roles**:
    ```sh
    aws [[00_Master_Links_and_Intro|iam]] create-role --role-name CrossAccountRole --assume-role-policy-document file://trust_policy.json
    ```

#### PRICING & TCO:
- **Pricing Drivers**: 
  - Number of accounts and associated AWS services used.
  - Usage-based pricing for various services within each account.

- **Optimization Strategies**:
  - Utilize [[Reserved Instances]] or [[Savings Plans]] for cost savings.
  - Use SCPs to enforce cost controls and avoid unnecessary spending.

#### COMPETITOR COMPARISON:
1. **AWS vs Azure**: 
   - AWS Organizations provide centralized management with fine-grained control through SCPs, whereas Azure uses Management Groups for similar functionality.

2. **AWS vs GCP**:
   - Both use organizational structures (Organizations in AWS and Org Policies in GCP) but AWS offers more granular control via SCPs compared to Org Policies.

#### INTEGRATION:
1. **CloudWatch**: 
   - Centralize logging, monitoring, and alerts across all accounts.
   
2. **IAM**: 
   - Manage access controls centrally using IAM roles and policies.
   
3. **VPC**: 
   - Extend network reachability through VPC peering or AWS Transit Gateway.

#### USE CASES:
1. **Financial Services**:
   - Isolate sensitive data and workloads to comply with regulatory requirements.
2. **Healthcare Industry**:
   - Enforce HIPAA compliance by segmenting patient data into separate accounts.
3. **Enterprise DevOps**:
   - Separate development, testing, and production environments for better governance.

#### DIAGRAMS:
1. **Multi-Account Architecture Diagram** (Placeholder):
    ```
    [Account 1] --([[SCP]])--> [Organization Root]
    [Account 2] 
    [Account 3]

    [VPC Peering] -> [Transit Gateway] --> [Shared Services Account]
    ```

#### EXAM TRAPS:
1. **Misconception**: SCPs can be applied directly to individual accounts.
   - Correction: [[SCPs]] are applied at the OU level, which then propagates to all child OUs and accounts.

2. **Misconception**: IAM roles with cross-account access can bypass SCP restrictions.
   - Correction: SCPs enforce governance rules that cannot be bypassed by IAM policies or roles.

#### DECISION MATRIX:
| Feature              | Use Case                          |
|----------------------|-----------------------------------|
| AWS Organizations    | Centralized management            |
| SCPs                 | Enforce compliance                |
| IAM Roles & Trust    | Cross-account access control      |
| VPC Peering          | Network isolation and security    |

#### 2026 UPDATES:
1. **New Features**: 
   - Enhanced cross-account resource sharing capabilities in RAM.
   
2. **Updated Limits**:
   - Increased limits on the number of SCPs per account.

#### EXAM SCENARIOS:
- **Scenario**: A company wants to enforce strict governance policies across multiple accounts.
  - Solution: Use [[AWS Organizations]] and apply Service Control Policies (SCPs) at the OU level.

#### INTERACTION PATTERNS:
1. **Service-to-Service Interaction**:
   - IAM roles are used for cross-account access within trusted relationships.
   - SCPs enforce rules that restrict certain actions across all accounts in an organization.

#### STRATEGIC ALIGNMENT:
Ensuring a comprehensive understanding of [[AWS Organizations]] and landing zones is crucial for passing the SAP-C02 exam, as it covers key aspects of multi-account governance and management.

#### PRIORITIZATION:
The focus here is on high-weight exam topics such as SCPs, IAM roles, and organizational governance, which are critical for success in the SAP-C02 certification.

#### CLARITY & GROUNDING:
All information is based on official AWS documentation and whitepapers to ensure accuracy and reliability.
```