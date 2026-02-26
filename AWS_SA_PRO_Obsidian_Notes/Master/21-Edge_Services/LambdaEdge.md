## [[RDS_Instance_Types|1. Advanced Architecture]]

[[Lambda@Edge]] is a variant of [[lambda|AWS Lambda]] that allows you to run Node.js and Python code in AWS edge locations, closer to your users, thus reducing latency and enabling a faster user experience. It extends the serverless capabilities of [[lambda]] by allowing developers to customize content delivered via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions.

Internally, [[Lambda@Edge]] operates by associating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] with specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] events, such as Viewer Request, Origin Request, Origin Response, and Viewer Response. The invocation mode determines when a function gets triggered based on these events.

Global scale is achieved through AWS's globally distributed edge network. Each edge location has its own set of resources, including compute capacity for running [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. This setup enables low-latency execution of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], even at high request rates.

Under the hood, [[Lambda@Edge]] uses Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets to store function packages (code and dependencies) and configuration metadata. Function versions or aliases can be specified during association with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service          | Use Case                                                              |
|-----------------|----------------------------------------------------------------------|
| [[Lambda@Edge]]    | Content manipulation, redirection, A/B testing, [[api-gateway|authentication]]       |
| [[api-gateway|API Gateway]]     | RESTful APIs, WebSocket APIs, HTTP APIs                               |
| [[appsync]]         | GraphQL APIs, real-time data synchronization                          |
| [[Srinivas_Notes/S3|S3]]              | Static website hosting, file storage, object retrieval               |

### Anti-Patterns

* Performing heavy processing tasks instead of using APIs or services designed for that purpose.
* Managing state within functions since [[Lambda@Edge]] doesn't support environment variables.
* Implementing complex [[api-gateway|authentication]] systems without considering pre-built solutions like AWS [[cognito]].

## [[RDS_Instance_Types|3. Security & Governance]]

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/lambda-edge-cloudfront"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction",
      "Condition": {
        "StringEquals": {
          "AWS:SourceVpc": "vpce-1234abcd-efgh-ijkl-mnop-qrstuvwx"
        }
      }
    }
  ]
}
```

### Cross-Account Access

To allow cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and attach a permission policy granting necessary actions to the target account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] service principal. In the target account, create a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] origin access identity (OAI) and update the source account's distribution with the OAI's origin access identity.

### Organization SCPs

Use [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to enforce restrictions on resource creation and modification across member accounts. For example, restrict the usage of specific AWS services or impose constraints on resource types.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the type of event triggering the [[lambda]] function. For example, viewer requests have a limit of 1,000 requests per second per function version. To handle higher traffic levels, implement exponential backoff strategies using client-side retry logic.

For high availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]), distribute the workload across multiple regions by creating separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions with different [[Lambda@Edge]] functions. Alternatively, use geo-routing to define custom routing rules based on the user's location.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include monitoring and configuring alerts based on [[billing|AWS Budgets]] and AWS [[billing|Cost Explorer]]. Additionally, apply [[organizations|AWS Organizations]] [[cost-allocation-tags|cost allocation tags]] to track costs across multiple accounts.

Calculate [[Lambda@Edge]] costs as follows:

* Number of invocations \* Execution time \* Request rate price (per 1M requests)
* Memory used \* Duration \* Price (per GB-second)
* Data transfer fees based on the amount of data transferred between AWS services and the internet

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

Scenario 1: A company wants to perform image optimization (resize and compression) on their website. They currently serve images from an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket via a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution. How would you optimize this solution using [[Lambda@Edge]]?

Correct answer: Configure [[Lambda@Edge]] to intercept Viewer Requests and modify the URL request to point to another [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution that triggers a [[lambda]] function responsible for resizing and compressing images before serving them.

Incorrect answer: Modify the existing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution to trigger a [[Lambda@Edge]] function directly. This approach may introduce additional latency due to function execution and increase complexity compared to using a separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution.

Scenario 2: A media organization wants to securely share video files stored in an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket with partner [[organizations]] while maintaining control over who can access the content. How would you achieve this goal using [[Lambda@Edge]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]?

Correct answer: Create a [[Lambda@Edge]] function associated with a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution that checks incoming requests against a list of authorized partners. If the request is valid, rewrite the URL to point to the appropriate [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. Otherwise, return an error message.

Incorrect answer: Implement a custom authorization system using [[Lambda@Edge]] to manage access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. This approach introduces unnecessary complexity and [[appsync|security]] risks compared to using AWS [[cognito]] or AWS WAF.