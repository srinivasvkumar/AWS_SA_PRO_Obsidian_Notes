**Advanced Architecture**

[[cloudformation]] Stack [[RDS_Instance_Types|Internals]]:

* A stack is an isolated collection of AWS resources that can be managed as a single unit.
* Stack creation using templates (JSON or YAML) which declare the desired resources and their configurations.
* [[cloudformation|Nested stacks]] allow managing child [[cloudformation|stacks]] within parent [[cloudformation|stacks]], enabling resource sharing and reusability.
* [[cloudformation|Change sets]] enable previewing updates before applying them, reducing potential downtime and [[api-gateway|errors]].

[[RDS_Instance_Types|Global Scale Considerations]]:

* [[cloudformation]] does not directly support multi-region deployments, but [[cloudformation|nested stacks]] and exported [[cloudformation|outputs]] help distribute resources across regions.
* Drift detection helps ensure consistency between actual and template-described resources in distributed systems.

Under The Hood Mechanics:

* Stack instances: Per-region [[api-gateway|caching]] of template configuration data for faster deployment and updates.
* Service roles: Allow [[cloudformation]] to create and manage resources on your behalf without exposing long-term credentials.

**Comparison & Anti-Patterns**

| Issue | Alternatives |
| --- | --- |
| Limited [[cloudformation|custom resources]] | Custom scripts, Lambda-based solutions, or third-party tools like Troposphere |
| Inadequate change management | [[CodeCommit]], CodeBuild, and CodePipeline for version control and continuous delivery |
| Tight coupling | Serverless Application Model ([[sam]]) or infrastructure-as-code tools (Terraform, Azure Resource Manager, Google Cloud Deployment Manager) |

Anti-patterns:

* Using [[cloudformation]] for transient resources (e.g., [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]])
* Mixing different resource types in a single template when multiple templates would provide better separation of concerns

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account Access:

* AssumeRole or GetCallerIdentity to delegate permissions from one account to another
* Service-linked roles for AWS services to call [[cloudformation]] APIs on your behalf

Organization SCPs:

* Prevent unauthorized resource creation by attaching Deny rules at the organization level
* Enforce tagging [[policies]] and budget constraints through SCPs

**Performance & Reliability**

Throttling Limits:

* Monitor throttling exceptions in [[cloudwatch]] metrics and implement exponential backoff strategies
* Adjust stack creation rates based on the number of resources and dependencies

HA/DR Patterns:

* Design stateless applications with auto scaling groups, load balancers, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] read replicas
* Implement multi-region [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] using [[route53]] [[route53|latency-based routing]] and [[dynamodb|DynamoDB global tables]]

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:

* Tag resources to track costs and enforce budget constraints
* Use [[billing|AWS Budgets]], [[billing|Cost Explorer]], and Trusted Advisor to optimize spending

Calculation Examples:

* Estimate monthly costs: `(Number_of_Resources * Cost_per_resource) + DataTransfer_costs`

**Professional Exam Scenarios**

Scenario 1: An enterprise has two AWS accounts: Account A for production workloads and Account B for development. They want to share resources between these accounts securely while maintaining strict governance. How should they configure [[cloudformation]]?

Correct Answer:

* Create cross-account role in Account A with necessary permissions
* Attach the role to the [[cloudformation]] service in Account B
* Use the assumed role in Account B to create and update resources in Account A

Incorrect Answer:

* Share resources directly between accounts without proper role-based access control
* Store long-term credentials in Account B for direct API calls to Account A

Scenario 2: A company needs to deploy a highly available web application with a relational database in three regions. The solution must minimize latency and cost. Which architecture pattern best fits their requirements?

Correct Answer:

* Active-active multi-region setup using [[route53]] [[route53|latency-based routing]], [[ALB]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] read replicas
* Use [[cloudformation]] [[cloudformation|nested stacks]] for consistent deployment and configuration of resources across regions

Incorrect Answer:

* Active-passive multi-region setup with manual failover and [[dr]] procedures, leading to increased latency during switchover and higher maintenance costs due to idle resources