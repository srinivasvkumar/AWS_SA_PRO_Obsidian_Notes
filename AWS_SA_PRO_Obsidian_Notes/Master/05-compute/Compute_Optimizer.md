### Advanced Architecture

At its core, Compute Optimizer is a service that analyzes the running resources in your environment and provides recommendations to improve the performance and cost of those resources. The service utilizes machine learning algorithms and cloud economics principles to analyze historical usage data and determine optimal [[ec2]] instance types or [[asg]] settings.

The [[RDS_Instance_Types|internals]] of Compute Optimizer involve three main components: the Data Ingestion Service, the Recommendation Engine, and the User Interface. The Data Ingestion Service collects metadata from [[cloudwatch]], Trusted Advisor, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]], while the Recommendation Engine uses the collected data to generate recommendations. The User Interface allows users to view and interact with these recommendations.

Global scale is achieved through regional endpoints, which can be used to query Compute Optimizer for recommendations. The service supports cross-region queries, allowing users to analyze resource usage across multiple regions. Under the hood, Compute Optimizer uses a combination of Amazon Elasticsearch, [[lambda|AWS Lambda]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] to provide these features.

### Comparison & Anti-Patterns

| Service | Use Case |
| --- | --- |
| Compute Optimizer | Improving the performance and cost of [[ec2]] instances and Auto Scaling Groups. |
| [[billing|Cost Explorer]] | Analyzing cost trends and identifying optimization opportunities. |
| Application Auto Scaling | Automatically scaling applications based on predefined rules. |

Anti-patterns include using Compute Optimizer as a standalone tool for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]], without considering other factors such as application architecture and business requirements. Additionally, assuming that all recommendations provided by Compute Optimizer should be implemented without proper validation could lead to unexpected issues.

### [[appsync|Security]] & Governance

To ensure secure access to Compute Optimizer, users can create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that allow or deny specific actions. For example, the following JSON policy snippet allows a user to list and get recommendations from Compute Optimizer:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "compute-optimizer:ListRecommendations",
                "compute-optimizer:GetRecommendationSummary"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-account access is supported through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]], allowing users in one account to assume an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in another account and perform actions on their behalf. To enforce organization-wide [[policies]], administrators can use Service Control [[policies]] (SCPs) to restrict or allow specific actions within Compute Optimizer.

### Performance & Reliability

Compute Optimizer has throttling limits in place to prevent overuse and maintain performance. If requests exceed the limit, the service will return a `ThrottlingException`. To handle this exception, users can implement exponential backoff strategies, such as doubling the delay between retries after each failure.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns can be implemented using multiple regions and ensuring that recommendations are replicated across them. This can be done by manually copying recommendations or using automation tools to synchronize them.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved by configuring Compute Optimizer to only analyze specific tags or resources. This ensures that recommendations are targeted towards specific workloads, reducing noise and improving relevancy.

For example, let's say you have two [[ec2]] instances, `web-server-1` and `db-server-1`, and you want to optimize the cost and performance of the web server but not the database server. You can apply a tag called `Optimize` with a value of `true` to the web server instance, and then configure Compute Optimizer to only analyze resources with that tag.

Here's an example of how to do this in the Compute Optimizer console:

1. Navigate to the Compute Optimizer Console.
2. Click on "Settings" in the left navigation pane.
3. Under the "Resources" section, click on "Add filter".
4. Select "Tag key" from the dropdown menu.
5. Enter `Optimize` as the tag key.
6. Set the tag value to `true`.
7. Click "Save changes".

This configuration will ensure that Compute Optimizer only considers the `web-server-1` instance when generating recommendations.

### Professional Exam Scenario 1

You are working as a solutions architect for a media company with a large presence in the US East (N. Virginia) region. Your team manages several hundred [[ec2]] instances and Auto Scaling Groups, and you have been tasked with optimizing the cost and performance of these resources. Which service would you recommend using?

Correct Answer: Compute Optimizer

Justification: Compute Optimizer is designed specifically for optimizing the cost and performance of [[ec2]] instances and Auto Scaling Groups. It uses machine learning algorithms and cloud economics principles to analyze historical usage data and determine optimal [[ec2]] instance types or [[asg]] settings.

Incorrect Answer: [[billing|Cost Explorer]]

Justification: While [[billing|Cost Explorer]] is useful for analyzing cost trends and identifying optimization opportunities, it does not provide specific recommendations for optimizing [[ec2]] instances or Auto Scaling Groups like Compute Optimizer does.

### Professional Exam Scenario 2

Your organization has strict [[appsync|security]] [[policies]] and requires that all AWS services must support cross-account access. Can Compute Optimizer be used in this scenario?

Correct Answer: Yes

Justification: Compute Optimizer supports cross-account access through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]], allowing users in one account to assume an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in another account and perform actions on their behalf. Administrators can also enforce organization-wide [[policies]] using Service Control [[policies]] (SCPs) to restrict or allow specific actions within Compute Optimizer.