```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Architecture Decision Records (ADRs)]]

#### UNDER-THE-HOOD MECHANICS:
[[Architecture Decision Records]] (ADRs) are a documentation practice to record the reasoning behind architectural decisions in software projects. ADRs typically consist of:

1. **Context**: The background and rationale for making the decision.
2. **Decision**: The actual choice or solution adopted.
3. **Status**: Whether the decision is provisional, accepted, deprecated, etc.
4. **Consequences**: Implications of this decision, both positive and negative.

The mechanics involve tracking changes over time through version control systems (e.g., [[Git]]). Each ADR is stored in a specific directory (commonly `adr/` or `docs/architecture/`) with a naming convention that includes the date and a short title.

#### EXHAUSTIVE FEATURE LIST:
- **Version Control Integration**: Automatically tracks changes.
- **Decision Tracking**: Provides historical context for architectural decisions.
- **Standardized Format**: Ensures consistency across all ADRs (e.g., [[Markdown]], JSON).
- **Searchable Documentation**: Facilitates finding relevant past decisions quickly.
- **Cross-Referencing**: Links related ADRs to provide comprehensive views of the architecture.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Set Up Repository**:
   - Initialize a Git repository for your project if it doesn’t already exist.
2. **Create Directory Structure**:
   - Create a directory (e.g., `adr/`) within the repository for storing ADRs.
3. **Define Template**:
   - Establish a template file (e.g., `template.md`) that includes placeholders for context, decision, status, and consequences.
4. **Write First ADR**:
   - Document an initial architectural decision using the template.
5. **Commit and Push**:
   - Commit changes to the repository and push them to your remote.

#### TECHNICAL LIMITS:
- **File Size**: Each ADR should be concise but detailed enough; typically, a limit of 1KB to 20KB is practical.
- **Version Control**: Ensures consistency and traceability in decision-making.

#### CLI & API GROUNDING:
While ADRs themselves do not have direct AWS CLI commands or APIs, the underlying version control system (e.g., [[Git]]) can be managed using tools like `git`:

```bash
# Clone repository
git clone <repository-url>

# Add new ADR file
echo "## 001-InitialDecision.md" > adr/001-InitialDecision.md

# Commit changes
git add .
git commit -m "Add initial architectural decision"
git push origin main
```

#### PRICING & TCO:
- **Version Control Costs**: Typically free for public repositories on [[GitHub]]/[[GitLab]]; private repositories have associated costs.
- **Hosting**: Cost of hosting the repository (e.g., [[cicd|AWS CodeCommit]], GitHub).

Optimization strategies include using free tier options and managing access controls to minimize costs.

#### COMPETITOR COMPARISON:
- **Design Docs in Google Cloud**: Similar concept but more focused on design than architecture decisions.
- **Wiki Pages**: More informal and less structured compared to ADRs.

Trade-offs involve the level of formality, ease of integration with version control systems, and long-term maintainability.

#### INTEGRATION:
ADRs can integrate seamlessly with [[00_Master_Links_and_Intro|other AWS services]] like [[iam]] for access controls and [[cloudwatch]] for monitoring repository activity. For example:

- **[[00_Master_Links_and_Intro|IAM]]**: Define roles and [[policies]] to restrict who can commit ADRs.
- **[[cloudwatch]]**: Monitor Git events (e.g., pushes) for compliance checks.

#### USE CASES:
1. **Large-Scale Systems**: Tracking decisions across multiple microservices.
2. **Enterprise Environments**: Documenting regulatory-compliant architecture choices.
3. **Open Source Projects**: Ensuring transparency and traceability in decision-making processes.

#### DIAGRAMS:
- Placeholder for a typical directory structure:

```
project-root/
├── adr/
│   └── 001-initial-decision.md
├── src/
└── README.md
```

#### EXAM TRAPS:
> [!danger] Distractor
Misunderstanding the purpose of ADRs vs. regular documentation.
Overlooking the importance of context and consequences in decision records.

#### DECISION MATRIX:
| Feature          | Use Case                       |
|------------------|--------------------------------|
| Version Control  | Large-scale projects           |
| Standardized Format | Enterprise compliance        |
| Cross-referencing | Complex, interconnected systems|

#### 2026 UPDATES:
- **Enhanced Integration**: Improved integration with cloud-native tools.
- **AI/ML Enhancements**: Automated suggestions based on past decisions.

#### EXAM SCENARIOS:
Questions may involve identifying the correct format for an ADR or recognizing the benefits of using ADRs over other forms of documentation.

#### INTERACTION PATTERNS:
ADRs interact primarily with version control systems and can be integrated into [[cicd|CI/CD]] pipelines for automated compliance checks.

#### STRATEGIC ALIGNMENT:
Understanding ADRs is crucial for managing complex architectural decisions, ensuring traceability, and maintaining consistency across teams.

### DistraCTOR ANALYSIS

**Scenarios Where This Service is a "Wrong" Answer:**
- **Scenario 1**: If your primary requirement is for serverless computing and you are looking to deploy stateless applications without managing servers, [[lambda|AWS Lambda]] would be more appropriate than Amazon [[00_Master_Links_and_Intro|EC2]].
- **Scenario 2**: For data warehousing purposes, [[redshift|Amazon Redshift]] is better suited compared to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] does not have the querying capabilities needed for complex analytical workloads.
- **Scenario 3**: If you need a managed database service that supports relational queries and transactional consistency, AWS [[00_Master_Links_and_Intro|RDS]] would be a more suitable choice than [[dynamodb]] which is optimized for NoSQL data storage.
- **Scenario 4**: For load balancing across multiple regions, Global Accelerator might be more appropriate compared to [[00_Master_Links_and_Intro|Application Load Balancer (ALB)]] which is typically used within a single region.
- **Scenario 5**: If your use case involves content delivery and [[api-gateway|caching]] globally, Amazon [[00_Master_Links_and_Intro|CloudFront]] would be preferable over AWS [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] as it provides optimized performance and reduced latency.

### THE "MOST CORRECT" LOGIC

**Trade-offs (e.g., Performance vs. Cost):**
- **AWS [[00_Master_Links_and_Intro|RDS]] vs. [[dynamodb]]**: [[00_Master_Links_and_Intro|RDS]] is a managed relational database service that offers ACID compliance, which can be critical for transactional workloads but comes at the cost of higher operational complexity and potential performance bottlenecks. [[dynamodb]], on the other hand, offers high scalability and low-latency reads/writes with less overhead in terms of maintenance but may not support complex query patterns.
- **[[lambda|AWS Lambda]] vs. [[00_Master_Links_and_Intro|EC2]]**: [[lambda|AWS Lambda]] is serverless and ideal for event-driven architectures where you don't want to manage infrastructure, offering cost savings when there are no executions. However, it has [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] on execution time (15 minutes max). [[00_Master_Links_and_Intro|EC2]] instances provide more control over the environment but require ongoing management of servers.

### ENTERPRISE GOVERNANCE

**Layers for [[organizations|AWS Organizations]], SCPs, [[ram]], and [[00_Master_Links_and_Intro|CloudTrail]]:**
- **[[organizations|AWS Organizations]]**: Use this to manage multiple accounts under a single entity, allowing you to apply consistent [[policies]] across all your [[AWS]] accounts.
- **Service Control [[policies]] (SCPs)**: Implement SCPs to restrict actions that can be performed within member accounts of an organization. For instance, prevent deletion of critical resources or limit access to certain services.
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Use [[ram]] to share AWS resources across multiple AWS accounts without the need for additional [[00_Master_Links_and_Intro|IAM]] roles or permissions.
- **AWS [[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to log management events and API calls made by users and AWS services. This helps in auditing actions taken within your environment, providing an audit trail of changes.

### TIER-3 TROUBLESHOOTING

**Complex Failure Modes (e.g., [[kms]] Key Policy Conflicts):**
- **[[kms]] Key Policy Conflicts**: Misconfigured [[kms]] key [[policies]] can lead to unauthorized access or denial of access issues. Ensure that [[00_Master_Links_and_Intro|IAM]] roles and users have the correct permissions to use a specific [[kms]] key.
- **[[cloudformation]] Stack Drift**: Resources in your stack might drift from their defined state due to manual changes outside [[cloudformation]] management, leading to inconsistencies.

### DECISION MATRIX

**Use Case vs. Solution Table:**

| Use Case                | Recommended Service        |
|-------------------------|----------------------------|
| Stateless Application    | [[lambda|AWS Lambda]]                 |
| Data Warehousing         | [[redshift|Amazon Redshift]]             |
| Managed Relational DB    | AWS [[00_Master_Links_and_Intro|RDS]]                    |
| Global Load Balancing    | [[SAP-C02_Vault/AWS Global Accelerator|AWS Global Accelerator]]     |
| Content Delivery Network | Amazon [[00_Master_Links_and_Intro|CloudFront]]          |

### RECENT UPDATES

**GA Features and Deprecated Items for 2026:**
- **[[lambda|AWS Lambda]] Layers**: As of 2026, expect ongoing enhancements in function deployment with layers, including support for more programming languages and simplified dependency management.
- **Deprecation Alert**: AWS will likely deprecate older [[00_Master_Links_and_Intro|EC2]] instance types that are no longer cost-effective or do not meet modern performance standards.