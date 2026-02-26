### Advanced Architecture

At its core, Amplify Studio is a frontend UI development tool that provides pre-built components, enabling faster web application development. The underlying architecture consists of several AWS services like [[appsync]], [[cognito]], [[lambda]], and [[cloudformation]]. A typical Studio project includes an Amplify Console hosting environment, Amplify Client libraries, and backend resources created using the Amplify CLI or via infrastructure as code (IaC) templates.

Studio supports multi-account deployments by allowing developers to create separate backends in different AWS accounts while maintaining a unified frontend configuration. This enables advanced architecture designs such as centralized [[api-gateway|authentication]], fine-grained resource sharing, and customized [[appsync|security]] [[policies]] per account.

#### [[RDS_Instance_Types|Global Scale Considerations]]

Global scale can be achieved through Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] CDN integration with Amplify Console. By default, static assets are distributed globally using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] edge locations. For dynamic content, developers can configure their own [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions and leverage [[lambda|AWS Lambda]]@Edge functions for serverless processing at the edge.

### Comparison & Anti-Patterns

| Service                     | When to use                                                                           | When not to use                                                               |
| ----------------------------| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Amplify Studio             | MVP/Prototype, internal tools, single page apps, mobile app backends              | Mission-critical applications requiring high availability and performance   |
| [[AWS CDK]]                    | Customizable infrastructure, Infrastructure management, DevOps teams            | Rapid prototyping, low-code/no-code projects, non-technical users          |
| AWS Serverless Application | Large-scale, event-driven serverless applications, long-term maintenance         | Projects requiring traditional VM/container deployment, monolithic apps      |

Anti-patterns include overusing Studio for mission-critical applications without considering potential scalability [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] and [[appsync|security]] vulnerabilities. Another common mistake is neglecting proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions setup, leading to unintended access to other AWS resources.

### [[appsync|Security]] & Governance

To establish cross-account access using Amplify Studio, you need to set up appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships between the participating AWS accounts. Here's an example JSON policy snippet granting necessary permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_B:root"
      },
      "Action": [
        "appsync:GraphQLGetMappingTemplate",
        "appsync:GraphQLGetSchemaCreationTime",
        "appsync:StartSchemaCreation",
        "appsync:UpdateGraphQLApi"
      ],
      "Resource": "arn:aws:appsync:REGION:ACCOUNT_A:apis/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpce": "vpce-1234567890abcdef0"
        }
      }
    }
  ]
}
```

Additionally, [[organizations]] can enforce granular control over Studio usage by implementing Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]].

### Performance & Reliability

Amplify Studio relies on various throttling limits depending on the underlying services used. To ensure high availability and prevent failures, implement exponential backoff strategies when handling [[api-gateway|errors]] during API calls or network connectivity issues.

For horizontal scaling and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes, distribute your application across multiple regions by utilizing Amplify Console environments and Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented by monitoring and adjusting Amplify Console build settings, [[api-gateway|caching]] [[policies]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution configurations. Additionally, optimize costs by selecting specific AWS Regions and services based on your application requirements.

Calculate cost estimates using the [AWS Pricing Calculator](https://calculator.aws/#/) and monitor consumption trends using AWS [[billing|Cost Explorer]].

### Professional Exam Scenarios

#### Scenario 1: Multi-Account Deployment

Suppose you want to deploy an Amplify Studio project across two AWS accounts for better resource isolation and governance. Which of the following approaches would ensure secure cross-account communication?

A. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user in Account A with the necessary [[appsync]] permissions, then share the [[appsync]] API with the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user from Account B.
B. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A with the necessary [[appsync]] permissions, add the ARN of Account B as a trusted entity, and attach the role to the [[appsync]] API.
C. Modify the [[appsync]] API in Account A to allow cross-origin requests from Account B.

Correct Answer: B

Justification: Option B is the most secure approach since it allows temporary assumption of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with restricted permissions. This minimizes the risk of unauthorized access compared to option A, which involves sharing the entire [[appsync]] API. Option C is incorrect because it does not address cross-account communication but rather focuses on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cross-Origin Resource Sharing (CORS)]] configuration.

#### Scenario 2: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]]

You have developed a web application using Amplify Studio with a single Amplify Console environment. How can you implement a robust [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] strategy?

A. Set up automatic backups for the Amplify Console environment and store them in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].
B. Implement a secondary Amplify Console environment in another region and use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] to route traffic between the primary and secondary environments.
C. Replicate the Amplify Studio project into a separate AWS account and enable [[appsync]] backup and restore functionality.

Correct Answer: B

Justification: Option B ensures high availability by distributing the application across multiple regions. If the primary environment fails, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] automatically reroute traffic to the secondary environment. Options A and C do not provide effective [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] mechanisms for the entire solution stack, including [[appsync]] and other integrated AWS services.