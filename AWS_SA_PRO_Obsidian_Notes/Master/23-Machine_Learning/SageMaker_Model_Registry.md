**Advanced Architecture: SageMaker Model Registry**

SageMaker Model Registry is a repository for model artifacts, allowing you to manage and share models across your organization. It integrates with other SageMaker features such as Pipelines, MPO (Model Package [[cloudformation|Outputs]]), and Studio to enable end-to-end machine learning workflows. The registry stores metadata about models, including training parameters, model performance metrics, and lineage information.

Internally, it uses Amazon [[dynamodb]] to store metadata, ensuring high availability and scalability. To support global scale, the underlying [[dynamodb]] table can be replicated across multiple regions using Global Tables. This enables low-latency access to model metadata from different locations while maintaining strong consistency.

When working with large [[organizations]] or multiple accounts, you can use [[organizations|AWS Organizations]] to centrally manage settings and apply Service Control [[policies]] (SCPs) to limit the use of the Model Registry. For example, you might restrict who can create, update, or delete models in the registry.

**Comparison & Anti-Patterns**

| Use Case | SageMaker Model Registry | Alternatives |
| --- | --- | --- |
| Centralized management of ML models | Yes | Store model artifacts in an [[Srinivas_Notes/S3|S3]] bucket and manually track metadata |
| Collaboration between teams | Yes | Share [[Srinivas_Notes/S3|S3]] buckets or use a separate metadata tracking system |
| Versioning and lineage tracking | Yes | Manually maintain versions and lineage data |
| Automated deployment pipelines | Partially | Integrate with CodePipeline and [[CodeCommit]] |

Anti-pattern: Using Model Registry only for versioning without taking advantage of its collaboration, sharing, and automation capabilities.

**[[appsync|Security]] & Governance**

To secure access to the Model Registry, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants permissions based on specific actions (e.g., `DescribeModelPackage`, `CreateModel`, etc.) and resources (model packages, models, or entire registries). Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:DescribeModelPackage",
                "sagemaker:CreateModel",
                "sagemaker:DescribeModel"
            ],
            "Resource": [
              "arn:aws:sagemaker:us-west-2:123456789012:registry/my-registry/*",
              "arn:aws:sagemaker:us-west-2:123456789012:model-package/my-registry/*"
            ]
        }
    ]
}
```

Cross-account access requires additional steps, such as creating a role in the source account, attaching an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy granting necessary permissions, and assuming the role in the destination account.

For multi-account scenarios, use [[organizations|AWS Organizations]] and apply Service Control [[policies]] (SCPs) to limit access to the Model Registry. For instance, you can prevent users from deleting models by denying the `sagemaker:DeleteModel` action.

**Performance & Reliability**

SageMaker Model Registry does not have built-in throttling limits. However, API calls to the underlying [[dynamodb]] table may be limited depending on your account settings. If you encounter throttling issues, implement exponential backoff strategies in your application code.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns depend on the underlying [[dynamodb]] table configuration. By default, [[dynamodb]] provides built-in replication and failover mechanisms. For more advanced [[dr]] scenarios, consider using point-in-time recovery (PITR) or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] to create backup plans and recover your [[dynamodb|DynamoDB tables]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls in the Model Registry are limited since it primarily relies on other SageMaker components like training jobs, hosting services, and storage. To optimize costs, focus on these areas instead:

1. Delete unused models, model packages, and registries to avoid unnecessary storage costs.
2. Implement lifecycle [[policies]] for Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets used by SageMaker to automatically remove older files.
3. Monitor usage patterns and adjust compute resources (ML instances, containers) accordingly.

Calculate costs using the AWS Pricing Calculator or the [[billing|Cost Explorer]] tool. Consider factors like the number of training jobs, model package sizes, and hours of model hosting when estimating costs.

**Professional Exam Scenarios**

Scenario 1: A pharmaceutical company wants to centralize their machine learning models across multiple accounts. They need to control access, ensure high availability, and monitor costs.

Correct answer: Use SageMaker Model Registry in combination with [[organizations|AWS Organizations]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], and [[dynamodb|DynamoDB Global Tables]]. Create a central Model Registry in one account and allow cross-account access via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Apply SCPs to limit access and usage. Enable [[dynamodb|DynamoDB Global Tables]] for low-latency access and failover capabilities. Monitor costs using AWS [[billing|Cost Explorer]].

Incorrect answer: Use individual Model Registries in each account and share metadata manually. This approach lacks centralized control, increases maintenance efforts, and doesn't provide cost monitoring capabilities.

Scenario 2: An e-commerce platform needs to train and deploy machine learning models at scale while maintaining high availability and managing costs.

Correct answer: Use SageMaker Model Registry along with other SageMaker features like Training Jobs, Hosting Services, and Pipelines. Implement multi-account strategies using [[organizations|AWS Organizations]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for centralized control and cost management. Ensure high availability through [[dynamodb|DynamoDB Global Tables]] and proper resource allocation.

Incorrect answer: Use a custom solution with manual model tracking, versioning, and deployment. This approach lacks integration with other SageMaker features, increases complexity, and doesn't provide centralized control or cost management capabilities.