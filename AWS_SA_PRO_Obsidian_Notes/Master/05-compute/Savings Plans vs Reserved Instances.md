```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[AWS Savings Plans]] vs. [[Reserved Instances]]

#### UNDER-THE-HOOD MECHANICS:
- **[[Savings Plans]]**:
  - Provides up to a 72% discount on usage.
  - Fixed pricing commitment over 1 or 3 years for specific resource categories (e.g., [[ec2]], [[Fargate]]).
  - Usage applied towards the plan reduces the cost of that usage; unused minutes are not credited back.

- **[[Reserved Instances (RIs)]]**:
  - Offers up to an 75% discount on usage.
  - Long-term commitment with upfront payment or partial upfront and hourly payments for remaining term.
  - Can be purchased in various types: No Upfront, Partial Upfront, All Upfront; Standard, Convertible, Scheduled RIs.

#### EXHAUSTIVE FEATURE LIST:
- **[[Savings Plans]]**:
  - Flexible on instance type, size, region, and even between services (e.g., [[ec2]] to [[Fargate]]).
  - Usage applied at a specific percentage (up to 100%) against the plan.
  - No upfront payment required.

- **[[Reserved Instances (RIs)]]**:
  - Fixed instance type and size.
  - Can be exchanged for other RIs in some cases with additional fees.
  - Option for partial or full upfront payments, reducing hourly costs significantly.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Evaluate Workload**: Assess your current workload and future usage patterns to determine the best savings plan type ([[Savings Plans]] vs [[RIs]]).
2. **Commitment Period Selection**: Choose between a 1-year or 3-year commitment.
3. **Purchase Plan/RIs**:
   - **For Savings Plans**: Navigate through AWS Management Console, select desired resource categories and commitment period.
   - **For RIs**: Select instance type, size, region, payment option (upfront/partial).
4. **Monitor Usage**: Use [[Cost Explorer]] to track usage against the plan or [[00_Master_Links_and_Intro|reserved instances]].

#### TECHNICAL LIMITS:
- **2026 Quotas**:
  - Savings Plans: No hard quotas but soft limits on total commitment amount.
  - RIs: Hard quota on number of reservations per account, flexible with instance type and size [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]].

#### CLI & API GROUNDING:
- **AWS CLI Commands**:
  ```bash
  # List all savings plans
  aws savingsplans describe-savings-plans

  # Purchase a new RI
  aws ec2 purchase-reserved-instances-offering --reserved-instances-offering-id <id>
  ```

- **API Flags**:
  - SavingsPlans: `DescribeSavingsPlans`, `CreateSavingsPlan`
  - RIs: `PurchaseReservedInstancesOffering`, `DescribeReservedInstances`

#### PRICING & TCO:
- **Cost Drivers**:
  - Usage patterns, commitment duration, upfront payments.
  
- **Optimization Strategies**:
  - Use [[billing|Cost Explorer]] for forecasting and tracking savings.
  - Right-size your instances to align with reserved capacities.

#### COMPETITOR COMPARISON:
- **[[AWS Savings Plans]] vs. [[Azure Reserved VMs]]**: AWS provides more flexible usage allocation across services; Azure is tied to specific services.
- **[[GCP Committed Use Discounts]] vs. RIs**: GCP requires commitment by region and machine type, less flexibility compared to AWS RIs.

#### INTEGRATION:
- **[[cloudwatch]]**: Both Savings Plans and RIs integrate with [[cloudwatch]] for monitoring usage and costs.
- **[[iam]]**: Permissions required for purchasing and managing both plans.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]**: Applies network configurations based on instance types within VPCs.

#### USE CASES:
- **Enterprise Example**:
  - Large E-commerce platform using [[ec2]], [[Fargate]], and [[rds]]. Savings Plans optimizes cross-service usage while RIs provide cost certainty for consistent workloads.

#### DIAGRAMS:
1. **Comparison Diagram**: Placeholder for diagram showing high-level comparison of Savings Plans vs. RIs.
2. **Integration Diagram**: Placeholder for integration with [[cloudwatch]], [[00_Master_Links_and_Intro|IAM]], [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

> [!abstract] Exam Tip
- Common Misconception: "RIs offer higher discounts than Savings Plans."
  - Clarification: Depending on usage patterns and commitment length, both can offer competitive discounts.

#### DECISION MATRIX:
| Features             | Use Cases                 |
|----------------------|---------------------------|
| Flexible Usage       | Cross-service workloads   |
| Long-term Commitment | Steady-state applications |
| Upfront Payments     | Budget flexibility        |

> [!warning] Quota
- 2026 Quotas: Savings Plans have soft limits on total commitment amount; RIs have hard quotas on reservations per account.

#### EXAM SCENARIOS:
- **Scenario**: You have an application that needs steady usage of [[ec2]] instances but also want flexibility for future workloads.
  - Solution: Recommend a combination of Standard RIs for steady-state and Savings Plans for flexible use cases.

> [!danger] Distractor
- Scenario 1: For short-term commitment (less than one year), On-Demand Instances are more suitable over Savings Plans or [[00_Master_Links_and_Intro|Reserved Instances]].

#### INTERACTION PATTERNS:
- Service-to-service interaction involves applying usage towards Savings Plans or [[Reserved Instances]] through Cost Management tools.

#### STRATEGIC ALIGNMENT:
- Focus on understanding key differences, benefits, and [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] for optimal workload alignment with AWS cost management services.

#### PROFESSIONAL TONE:
High-value delivery focusing on precise technical details and strategic recommendations tailored for SAP-C02 candidates.

#### AUDIENCE:
Tailored specifically for SAP-C02 professionals preparing for the exam.

#### PRIORITIZATION:
Focus on high-weight topics such as pricing models, commitment periods, and integration with Cost Management tools.

#### CLARITY:
Concise explanations for complex concepts ensuring clarity in understanding Savings Plans vs. RIs.

#### GROUNDING:
Reference official AWS documentation and whitepapers to validate information.

#### VALIDATION:
Align with the latest exam blueprint to ensure accuracy and relevance.

#### ARCHITECTURAL TRADE-OFFS:
Understanding the impact of design choices on implementation, such as flexibility versus cost savings, is crucial for effective planning.