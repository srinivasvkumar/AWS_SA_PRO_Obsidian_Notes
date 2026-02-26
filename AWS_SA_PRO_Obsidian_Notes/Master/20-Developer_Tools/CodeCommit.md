**[[RDS_Instance_Types|1. Advanced Architecture]]**

CodeCommit is a fully-managed source control service that hosts private Git repositories. It utilizes Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storage, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] for access control, and [[lambda]] for automating certain processes. CodeCommit uses a Merkle Tree structure for deduplication and integrity checks.

Global scale is achieved through regional replication of repositories in an account. However, there's no cross-region replication provided by CodeCommit itself. To achieve this, you need to implement custom solutions using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or other tools.

Internally, each repository consists of three main components:

- *Objects*: These are stored in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and include files, directories, and Git metadata like commit history.
- *Triggers*: [[lambda|AWS Lambda]] functions invoked when specific repository events occur (e.g., push, pull request creation).
- *Approvval Rules*: [[cloudformation|Conditions]] under which pull requests must be approved before merging.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| CodeCommit      | Version control within AWS ecosystem, especially with CodeBuild, CodeDeploy |
| GitHub          | Public or open-source projects, better [[cicd|CI/CD]] integration with non-AWS tools |
| Bitbucket Cloud | Similar to GitHub, also supports both cloud and server versions       |

Anti-pattern: Using CodeCommit solely because it's managed by AWS without considering if it fits your workflow requirements or team preferences.

**[[RDS_Instance_Types|3. Security & Governance]]**

CodeCommit allows managing permissions at different levels (repo, branch, or file) via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] written in JSON. For example:

```json
{
    "Effect": "Allow",
    "Action": ["codecommit:GitPush"],
    "Resource": "arn:aws:codecommit:*:123456789012:my-repository",
    "Condition": {
        "StringEquals": {"aws:SourceVault": "https://my-vault.s3.amazonaws.com"}
    }
}
```

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or resource-based [[policies]]. Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] across accounts.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

The default throttle limit for CodeCommit is 5,000 requests per user per second. If these limits are exceeded, exponential backoff should be implemented in client applications. High availability is inherent due to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]'s redundancy, but [[dr]] requires manual setup of backups and recovery plans.

**[[RDS_Instance_Types|5. Cost Optimization]]**

There are no additional charges for active repositories, making [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] straightforward. However, understanding how many repositories are needed and limiting unused ones helps prevent unnecessary complexity.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

*Scenario 1*: A company has two AWS accounts: one for production and another for staging. They want to share a single CodeCommit repo between them while maintaining strict access control. How would you set this up?

*Correct Answer*: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in each account allowing `codecommit:GitPush` and `codecommit:GitPull`. Attach trust relationships from the other account. Users in both accounts will then assume the appropriate role when interacting with the shared repo.

*Incorrect Answer*: Add users from both accounts directly to the CodeCommit repo. This violates the principle of least privilege and doesn't provide clear separation of concerns.

*Scenario 2*: A large enterprise needs to manage costs associated with several hundred repositories spread across multiple AWS accounts. What strategy could they adopt?

*Correct Answer*: Implement AWS Resource Groups Tagging API (RGTAPI) along with [[cost-allocation-tags|cost allocation tags]]. Apply consistent tagging [[policies]] across all accounts and regularly [[nonportable_instructions|review]] usage trends.

*Incorrect Answer*: Rely on AWS [[billing|Cost Explorer]] alone. While useful, it lacks granularity required to analyze specific services like CodeCommit.