**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[glue|AWS Glue]] Crawlers are data cataloging mechanisms that identify and describe data stored in various sources. They play a crucial role in ETL workflows by extracting metadata from diverse datasets, enabling data transformation and loading into a target system like Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[redshift]].

Internally, crawlers utilize classification methods based on schema detection, which can be customized using regular expressions or Python scripts. The crawler job then writes the gathered metadata to the [[glue|AWS Glue]] [[glue|Data Catalog]], making it accessible for querying or further processing.

For [[RDS_Instance_Types|global scale considerations]], AWS recommends deploying separate accounts for each region with necessary resources such as [[glue]] Crawlers created within these isolated environments. This strategy enhances [[appsync|security]], reduces potential blast radius during failures, and enables more precise cost management.

When scaling horizontally, keep in mind that every crawler invocation counts towards your account's concurrent running executions limit. If you reach this threshold, additional jobs will queue until capacity becomes available. To mitigate this, implement an appropriate retry mechanism using [[step-functions|AWS Step Functions]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] with exponential backoff.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service            | Use Case                                                              |
|-------------------|--------------------------------------------------------------------|
| [[glue]] Crawlers     | Automating schema discovery and metadata management for structured data.   |
| AWS DataBrew      | Interactive data [[Git_hub_notes/certified-aws-solutions-architect-professional-main/README|preparation]] for analytics without coding skills.    |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/08-data-analytics/others|AWS Lake Formation]]| Centralized management of data access, fine-grained permissions. |

Anti-pattern: Using [[glue]] Crawlers when real-time streaming is required. Instead, opt for services like AWS [[kinesis]] or MSK.

**[[RDS_Instance_Types|3. Security & Governance]]**

To ensure secure cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account allowing [[glue]] Crawler actions. Then, attach a trust policy permitting the destination account's crawler principal:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::DestinationAccountID:root"
      },
      "Action": [
          "glue:getCrawler*,glue:GetDatabase*,glue:GetTable*"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpce": "vpce-1234abcd"
        }
      }
    }
  ]
}
```
In the destination account, grant permissions to invoke the crawler through its ARN.

Organizational Service Control [[policies]] (SCPs) can enforce restrictions on specific operations at an organizational level. For instance, denying the creation of new crawlers would prevent unwanted costs or unauthorized data access.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits for [[glue]] Crawlers include 50 simultaneous crawls per account by default. To handle higher numbers, submit a support request to increase the quota.

Exponential backoff strategies should be implemented when dealing with throttled requests due to insufficient execution capacity. This technique involves waiting for progressively longer periods before retrying failed tasks.

HA/DR patterns may involve replicating metadata across multiple regions for faster [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]. However, this approach could lead to consistency issues if not properly managed.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved by setting up [[billing]] alerts based on usage thresholds and tagging all related resources. Additionally, consider using on-demand pricing for infrequent crawler usage rather than committing to reserved capacity.

Calculation example: Suppose a single crawl consumes 0.15 DBUs ([[glue]] Data Processing Units) per second. With a price of $0.000016 per DBU-second, the total cost for one minute of crawling is approximately $0.012.

**6. Professional Exam Scenario**

Scenario A: An organization wants to automate schema discovery and metadata management for their structured data stored in various [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets across different AWS accounts. What solution architecture would you propose?

Correct answer: Use [[glue|AWS Glue]] Crawlers in combination with [[organizations|AWS Organizations]] to enable centralized management and cross-account access. Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in each source account allowing [[glue]] Crawler actions, and trust [[policies]] permitting the destination account's crawler principal. Tag all involved resources for granular cost control.

Incorrect answer: Implement AWS DataBrew for automated schema discovery and metadata management. While DataBrew provides interactive data [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/README|preparation]] features, it does not natively support cross-account metadata synchronization.

Scenario B: A company operates numerous [[glue]] Crawlers that often exceed the concurrent running executions limit, causing delays in data processing workflows. How would you address this issue while maintaining high availability?

Correct answer: Implement an appropriate retry mechanism using [[step-functions|AWS Step Functions]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] with exponential backoff. By doing so, you ensure timely task completion even under high load [[cloudformation|conditions]].

Incorrect answer: Request an increase in the concurrent running executions limit without implementing any retry strategy. Although increasing the limit might provide temporary relief, it does not address the underlying problem of insufficient execution capacity during peak usage times.