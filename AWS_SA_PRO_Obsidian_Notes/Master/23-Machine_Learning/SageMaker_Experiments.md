**[[RDS_Instance_Types|1. Advanced Architecture]]: SageMaker Experiments**

SageMaker Experiments is a management feature of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] that enables you to organize, evaluate, and visualize machine learning (ML) experiments across multiple modeling jobs. It allows you to group and compare trials, view metrics side-by-side, and track parameters and metadata in a searchable form. The [[RDS_Instance_Types|internals]] of SageMaker Experiments involve tracking the following components:

- **Trials**: A collection of one or more training jobs, processing jobs, endpoints, or model packages. Trials can be created manually or via automation using Amazon [[cloudformation]], AWS SDKs, or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Cloud Development Kit (CDK)]].
- **TrialComponents**: Individual tasks within a trial such as specific hyperparameter tuning jobs, transform jobs, or endpoint configurations.
- **Metrics**: Quantitative values associated with each TrialComponent, such as accuracy, loss, or latency. Metrics can be automatically recorded from ML runs or manually added.

For [[RDS_Instance_Types|global scale considerations]], SageMaker Experiments supports integration with [[organizations|AWS Organizations]] allowing centralized management of resources and costs. To implement this, create an SageMaker Service Category and link it to an Organizational Unit (OU) or an entire organization. Then, apply a Service Control Policy ([[SCP]]) to enforce usage [[control-tower|guardrails]].

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Issue | SageMaker Experiments | Alternatives |
| --- | --- | --- |
| Limited Customization | Limited customization options for experiment reports and visualizations. | Custom notebooks or ML pipelines for tailored reporting and visualization. |
| Data Privacy | Sensitive data might be stored in experiment records. | Use customer-managed keys (CMKs) for encrypting experiment records. |
| Cost Management | Increased costs due to additional API calls and resource utilization. | Implement granular cost controls through [[cost-allocation-tags|cost allocation tags]], [[Budgets]], and alerts. |

Common design mistakes include mixing different types of trials or trial components within a single experiment, not setting appropriate metric thresholds, and failing to version control models or code.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for SageMaker Experiments can be implemented by attaching permissions directly to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles. Here's an example JSON policy snippet that grants `sagemaker:CreateExperiment`, `sagemaker:CreateTrial`, and `sagemaker:CreateTrialComponent` actions to a role named `MySageMakerRole`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:CreateExperiment",
                "sagemaker:CreateTrial",
                "sagemaker:CreateTrialComponent"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::123456789012:role/MySageMakerRole"
                }
            }
        }
    ]
}
```
Cross-account access to SageMaker Experiments can be achieved using AWS [[sts]] AssumeRole, enabling secure delegated access between accounts. For stricter governance, use Organization SCPs to limit SageMaker Experiments functionality at scale.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

To manage throttling limits and avoid [[api-gateway|errors]] like `ThrottlingException`, implement exponential backoff strategies using the AWS SDKs. This ensures your application retries requests with increasing delays until the request succeeds.

High availability (HA)/disaster recovery ([[dr]]) patterns can be implemented by creating separate ML experiment environments in multiple regions. Use [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] DNS routing [[policies]] to distribute traffic efficiently.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be applied by leveraging SageMaker pricing models (On-Demand, Spot, and Provisioned instances) based on workload requirements. Additionally, optimize costs by implementing instance size flexibility, terminating idle endpoints, and using AWS [[cost-allocation-tags|cost allocation tags]] to track spending.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

**Scenario 1:** Your company operates globally and wants to implement a centralized SageMaker Experiments solution for all its departments. How would you set up cross-account access, and how would you ensure governance?

*Implement cross-account access using AWS [[sts]] AssumeRole. Create a dedicated SageMaker Experiments account and attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with necessary permissions. In other accounts, grant permission to call `assume_role` action for the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the SageMaker Experiments account. Apply Service Control [[policies]] (SCPs) at the organizational level to restrict unwanted SageMaker Experiments actions.*

**Scenario 2:** You need to implement a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] strategy for SageMaker Experiments while maintaining performance and reliability. What steps would you take?

*Utilize SageMaker pricing models based on workload requirements. Leverage On-Demand instances for short-lived experiments, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]] for background workloads, and Provisioned instances for steady-state workloads. Implement instance size flexibility, terminate idle endpoints, and use AWS [[cost-allocation-tags|cost allocation tags]] to track spending. Ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] by creating separate ML experiment environments in multiple regions.*