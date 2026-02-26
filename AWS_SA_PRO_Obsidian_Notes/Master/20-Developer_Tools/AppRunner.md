**[[RDS_Instance_Types|1. Advanced Architecture]]**

AppRunner is a fully managed service that enables developers to quickly deploy containerized web applications. It provides automatic scaling, high availability, and a simple API to manage applications.

Internally, AppRunner uses a serverless infrastructure to run containers. Each application has one or more services, which can have multiple versions. Services are automatically load balanced across multiple Availability Zones (AZs) within a region. AppRunner creates an Application Boundary (AppBoundary) to isolate resources and ensure secure communication between services.

For global scale, you can create regional deployments in multiple regions and configure cross-region replication using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. This setup allows low-latency connections worldwide while maintaining data residency requirements.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| AppRunner            | Fast deployment of containerized web apps, minimal maintenance     |
| [[ecs]] ([[Fargate]])         | Customizable container management, fine-grained control             |
| [[lambda]]              | Serverless compute for event-driven functions                       |
| Batch                | Automated batch processing with flexible resource allocation        |

Anti-pattern: Using AppRunner for long-running background tasks. Instead, use [[lambda]], [[ecs]] with [[Fargate]], or Batch.

**[[RDS_Instance_Types|3. Security & Governance]]**

To enable cross-account access, set up an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account allowing `apprunner:StartVpcConnection` permissions. In the destination account, attach a policy granting the necessary [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] permissions.

Organization Service Control [[policies]] (SCPs):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "apprunner:Create*",
                "apprunner:Delete*",
                "apprunner:Update*"
            ],
            "Resource": "*"
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the number of active runs per account. To handle throttling, implement exponential backoff strategies using the AWS SDK. For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute applications across multiple AZs and regions.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include setting a monthly budget and configuring alerts. To optimize costs, rightsize your application by adjusting the minimum and maximum number of instances based on traffic predictions. Additionally, monitor idle resources and release them when not needed.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

**Scenario 1:** A company needs to deploy a containerized web app globally with low latency and strict data residency rules. They want to minimize operational overhead. Which solution meets these requirements?

*Correct Answer*: Use AppRunner with regional deployments and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] for global distribution. Configure origin access identities (OAI) in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to restrict direct access to AppRunner origins, ensuring data residency compliance.

*Incorrect Answers*:

* Deploying directly to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] without AppRunner would require managing the underlying infrastructure, increasing operational overhead.
* Using [[ecs]] ([[Fargate]]) requires additional configuration and management efforts compared to AppRunner.

**Scenario 2:** A consulting firm manages multiple accounts for its clients. The consultancy wants to enforce a [[appsync|security]] policy that prevents creating or updating any AppRunner resources.

*Correct Answer*: Implement an Organization [[SCP]] denying all `apprunner:Create*`, `apprunner:Delete*`, and `apprunner:Update*` actions.

*Incorrect Answers*:

* Modifying [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or instance profiles does not prevent creating or updating AppRunner resources since they can be created via APIs.
* Configuring Service Quotas does not prevent creating or updating AppRunner resources but only sets usage limits.