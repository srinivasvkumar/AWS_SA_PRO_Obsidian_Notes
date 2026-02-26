```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[Shared VPC Architecture]]

#### UNDER-THE-HOOD MECHANICS:

A [[Shared VPC]] architecture allows multiple projects or accounts to use a common [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] without needing their own network infrastructure. This is achieved through the following mechanics:

1. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Owner**: The account that owns the shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
2. **Service Project**: Accounts that consume services from the shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

The owner can allow other accounts (service projects) to attach subnets, route tables, and [[appsync|security]] groups to their resources within the shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] without granting full access to the entire network infrastructure. This design ensures better [[appsync|security]] by isolating service project resources into separate subnets or [[appsync|security]] groups while leveraging a single, centralized network.

#### EXHAUSTIVE FEATURE LIST:

- **Shared Subnet**: Service projects can attach specific subnets.
- **Service Project Association**: Allows associating multiple service projects with the shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
- **[[00_Master_Links_and_Intro|IAM]] Role and Policy Management**: Controls access through [[IAM roles]] and [[policies]].
- **Route Table Sharing**: Route tables can be shared, allowing fine-grained control over routing decisions.
- **[[appsync|Security]] Group Control**: Service projects can use or create [[appsync|security]] groups within the shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

#### STEP-BY-STEP IMPLEMENTATION:

1. **Create Shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**:
   - In the owner account, create a new [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]].
2. **Configure Sharing**:
   - Share specific subnets and route tables with service project accounts.
3. **[[00_Master_Links_and_Intro|IAM]] Role Setup**:
   - Set up [[00_Master_Links_and_Intro|IAM]] roles in both the owner and service project accounts to manage permissions.
4. **Attach Subnet and Route Table**:
   - Service projects attach their instances or resources to shared subnets and route tables.

#### TECHNICAL LIMITS (2026):

- A maximum of 100 service projects can be associated with a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
- Each subnet within the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] can be attached by up to 50 service project accounts.
- There are no limits on the number of subnets that can be shared.

#### CLI & API GROUNDING:

```sh
aws ec2 modify-vpc-attribute --vpc-id vpc-1234567890abcdef0 --enableSharedTenancy
```

**API Flags**:
- `ModifyVpcAttribute` for enabling shared tenancy.
- `AssociateSubnetCidrBlock` to associate CIDR blocks.

#### PRICING & TCO:

Cost drivers include the number of instances, storage, and data transfer within the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. Optimization strategies involve proper subnet utilization, reserved instance pricing, and cost allocation tagging.

#### COMPETITOR COMPARISON:

- **Google Shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Provides a similar functionality but with differences in quota limits and management through Google Cloud Console and APIs.
- **Azure Virtual Network Service Endpoints**: Azure provides service endpoints for secure access to services within a virtual network.

#### INTEGRATION:

- **[[cloudwatch]]**: Metrics can be collected for monitoring resource usage, [[appsync|security]] group activity, etc.
- **[[00_Master_Links_and_Intro|IAM]]**: [[00_Master_Links_and_Intro|IAM]] roles control permissions for accessing [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] resources.
- **[[vpc-flow-logs|VPC Flow Logs]]**: Capture detailed information about the IP traffic going to and from network interfaces in your shared subnets.

#### USE CASES:

1. **Multi-Account Environments**: Enterprises with multiple accounts can leverage a single, centrally managed [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
2. **Centralized [[appsync|Security]] Management**: Simplifies [[appsync|security]] management across various projects by using centralized [[policies]].

#### DIAGRAMS:

- Placeholder for a diagram showing the architecture of shared subnets and associated service projects within a shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

> [!abstract] Exam Tip:
Common misconception: Thinking that all resources in a shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] are automatically secure without proper [[00_Master_Links_and_Intro|IAM]] roles.
Misunderstanding the difference between shared tenancy and [[00_Master_Links_and_Intro|dedicated instances]].

#### DECISION MATRIX:

| Feature              | Use Case 1 (Multi-Account)          | Use Case 2 (Centralized [[appsync|Security]])    |
|----------------------|-------------------------------------|---------------------------------------|
| Subnet Sharing       | High                                 | Medium                                |
| [[00_Master_Links_and_Intro|IAM]] Role Management  | Medium                               | High                                  |
| Route Table Control  | Medium                               | Low                                   |

#### 2026 UPDATES:

- AWS has introduced new APIs and CLI commands to streamline the sharing process.
- Enhanced monitoring tools for better visibility into shared resources.

> [!check] Implementation:
Questions often involve configuring [[00_Master_Links_and_Intro|IAM]] roles, associating subnets with service projects, and ensuring compliance with [[appsync|security]] [[policies]].

#### INTERACTION PATTERNS:

Service-to-service interactions within a [[Shared VPC]] rely on the correct configuration of route tables, subnets, and [[appsync|security]] groups to ensure secure and efficient communication.

> [!warning] Quota:
- A maximum of 100 service projects can be associated with a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
- Each subnet within the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] can be attached by up to 50 service project accounts.

#### STRATEGIC ALIGNMENT:

Focus on understanding the mechanics, implementation steps, and use cases relevant to SAP-C02.

> [!danger] Distractor:
Scenarios where Shared [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is a "wrong" answer include environments requiring complete isolation (e.g., strict compliance mandates) or high-performance requirements that benefit from direct control over network infrastructure.