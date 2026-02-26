```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[AWS Backup]] Cross-Region & Cross-Account Deep-Dive

#### Under-the-Hood Mechanics:
[[AWS Backup]] allows the creation, management, and restoration of backups across regions and accounts by leveraging [[iam]] roles and permissions. The cross-region backup involves copying data from one region to another using the [[00_Master_Links_and_Intro|AWS Backup]] service itself or through [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] cross-region replication (CRR) if dealing with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] backups.

1. **Internal Data Flow**:
    - Data is first backed up in the source region.
    - A secondary process handles the transfer of this backup to a different region, either directly by [[AWS Backup]] or indirectly via [[S3 CRR]].
2. **Consistency Models**:
    - Point-in-time consistency ensures that all data within the same restore point reflects the state at a specific time.
    - Data integrity is maintained through checksums and encryption mechanisms during transit and at rest.
3. **Underlying Infrastructure Logic**:
    - Cross-region backups are managed by [[AWS Backup]], which orchestrates the transfer across regions using [[iam]] roles for access control.
    - The cross-account functionality leverages [[00_Master_Links_and_Intro|IAM]] roles with appropriate permissions to allow one account to manage resources in another account.

#### Exhaustive Feature List
1. **Cross-Region Backups**:
    - Automatic or manual copying of backups from source region to destination region.
2. **Cross-Account Backups**:
    - Ability to back up and restore resources across different [[AWS]] accounts, ensuring data portability and compliance with organizational [[policies]].
3. **[[00_Master_Links_and_Intro|IAM]] Role Management**:
    - Use of [[00_Master_Links_and_Intro|IAM]] roles for cross-account access control.
4. **Backup Plans & Rules**:
    - Customizable backup plans and rules to automate the backup process.
5. **Encryption Support**:
    - Encryption at rest using [[AWS KMS]] keys in both source and target regions/accounts.
6. **Cross-Account Restoration**:
    - Ability to restore backups from a different account.

#### Step-by-Step Implementation
1. **Create [[00_Master_Links_and_Intro|IAM]] Roles**
    ```sh
    aws iam create-role --role-name CrossAccountBackupRole --assume-role-policy-document file://trust-policy.json
    ```
2. **Attach [[policies]]**
    ```sh
    aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForBackup --role-name CrossAccountBackupRole
    ```
3. **Configure Backup Plan**
    - Define backup rules and plans using the [[AWS Management Console]] or CLI.
4. **Set Up Cross-Account Access**
    ```sh
    aws sts assume-role --role-arn arn:aws:iam::target-account-id:role/CrossAccountBackupRole --role-session-name BackupSession
    ```
5. **Initiate Backups**
    - Use [[AWS Backup]] to initiate backups from source account to target account/region.

#### Technical Limits (2026)
- Maximum number of backup vaults per region: 100.
- Number of recovery points per vault: Configurable up to a maximum defined by AWS.
- Soft quota for cross-account access [[policies]] may limit the number of roles and trust relationships you can create.

> [!warning] Quota
- Ensure not to exceed the soft quotas, which can impact your backup operations.

#### CLI & API Grounding
- **CLI Command**
    ```sh
    aws backup list-backup-vaults --region us-west-2
    ```
- **API Flags**
    - `ListBackupVaults`
    - `CreateBackupPlan`
    - `StartBackupJob`

> [!check] Implementation
Use CLI and API to manage your backup configurations effectively.

#### Pricing & TCO
- Cost based on storage size and number of recovery points.
- Additional costs for cross-region data transfer.
- Optimization strategies include using lifecycle [[policies]] to archive older backups.

#### Competitor Comparison
- **Azure Backup**: Supports cross-account but limited cross-region functionality without manual intervention.
- **Google Cloud Backup**: Limited cross-account support, requires more manual setup compared to [[AWS Backup]].

> [!abstract] Exam Tip
Understand the differences between [[00_Master_Links_and_Intro|AWS Backup]] and its competitors in terms of cross-region and cross-account capabilities.

#### Integration
- **[[cloudwatch]]**:
    - Integration for monitoring backup job status and alarms on failures.
- **[[00_Master_Links_and_Intro|IAM]]**:
    - Roles and [[policies]] for managing access across accounts and regions.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**:
    - Backups can be configured to operate within specific VPCs, ensuring network isolation.

#### Use Cases
1. **[[00_Master_Links_and_Intro|Disaster Recovery]]**
    - Critical systems backed up in a different region for [[dr]] purposes.
2. **Compliance & Audits**
    - Regulatory requirements necessitating cross-account data sharing and backup.

> [!danger] Distractor
Be cautious that [[00_Master_Links_and_Intro|AWS Backup]] is not designed for real-time or near-real-time replication of data, as it focuses on scheduled backups rather than continuous replication.

#### Decision Matrix

| Feature                 | Use Case                |
|-------------------------|-------------------------|
| Cross-Region Backups    | [[00_Master_Links_and_Intro|Disaster Recovery]]       |
| Cross-Account Backups   | Compliance Audits       |
| [[00_Master_Links_and_Intro|IAM]] Role Management     | Secure Access Control   |

#### 2026 Updates
- Enhanced support for cross-account backup and restore workflows.
- Improved integration with [[AWS Organizations]].

#### Exam Scenarios
1. **Scenario**: Design a [[00_Master_Links_and_Intro|disaster recovery]] strategy using cross-region backups.
2. **Question**: What are the steps to enable cross-account backups?

#### Interaction Patterns
- Service-to-service interactions include [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] CRR and [[00_Master_Links_and_Intro|IAM]] role assumption for cross-account access.

#### Strategic Alignment
- Focus on high-weight exam topics like [[00_Master_Links_and_Intro|IAM]] roles, backup plans, and cross-region/cross-account workflows.
- Justify each piece of information with real-world use cases relevant to SAP-C02 candidates.

> [!check] Implementation
Ensure you cover all critical exam topics comprehensively for a thorough understanding.