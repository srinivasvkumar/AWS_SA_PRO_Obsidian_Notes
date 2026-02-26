### Advanced Architecture
At its core, Amplify Hosting is a static web hosting service that integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], and [[Lambda@Edge]] to provide a seamless deployment and hosting experience for modern web applications. The following diagram provides an overview of the components involved in an Amplify Hosting setup:

```mermaid
graph TD
    A[Amplify Console] -->|Deploys| B(S3)
    B -->|Served via| C[CloudFront]
    D[Lambda@Edge] -- Add custom functionality --> C
    E[Custom Domain] --> C
    F[ACM] --> E
    G[Route 53] --> E
```

When deploying a static website using Amplify Hosting, here are some advanced architecture considerations:

* Multi-region deployments: To achieve low-latency connections worldwide, you can configure your Amplify app to deploy to multiple regions by creating separate branches for each region. This allows you to serve content from the nearest location to your users.
* Content transformation with [[Lambda@Edge]]: With [[Lambda@Edge]], you can perform serverless transformations on your site's assets at [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Edge locations. For example, you can manipulate images, inject user-specific data, or even modify HTML content.
* Custom domain configuration: You can connect your custom domains to your Amplify Hosting app using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Certificate Manager (ACM)]] to manage SSL certificates.

### Comparison & Anti-Patterns

Below is a table comparing Amplify Hosting to other AWS hosting services like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], and Elastic Beanstalk:

| Service            | Static Site Hosting | Serverless Functions | CDN Capabilities | Custom Domains |
|--------------------|---------------------|---------------------|-----------------|---------------|
| **Amplify Hosting** | Yes                | Limited             | Yes             | Yes          |
| **Amazon [[Srinivas_Notes/S3|S3]]**      | Yes                | No                  | No              | Yes          |
| **[[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]**     | No                 | No                  | Yes             | Yes          |
| **Elastic Beanstalk** | No                | Yes                | Yes             | Yes          |

Common anti-patterns when using Amplify Hosting include:

* Using Amplify Hosting for dynamic websites: Amplify Hosting is designed for static sites, so it may not be suitable for hosting dynamic websites without additional configurations.
* Not configuring proper cache settings: Incorrect cache settings might lead to stale or inconsistent content being served to end-users.

### [[appsync|Security]] & Governance

To secure your Amplify Hosting setup, you should follow these [[iam|best practices]]:

* Implement least privilege access: Create granular [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Amplify Console, [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to ensure only necessary permissions are granted.
* Use Organization Service Control [[policies]] (SCPs): Set up centralized control over [[organizations|AWS Organizations]] to enforce [[appsync|security]] rules across all member accounts.
* Enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cross-origin resource sharing (CORS)]]: Configure CORS settings to allow specific domains to interact with your application resources.

Example JSON policy granting Amplify Console access to create and update [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::*"
      ]
    }
  ]
}
```

### Performance & Reliability

To optimize performance and reliability for Amplify Hosting setups, consider the following:

* Throttling limits: Monitor throttling limits for Amplify Console and increase them if needed.
* Exponential backoff strategies: Implement exponential backoff strategies for API calls during periods of high traffic to prevent overwhelming the system.
* High availability/disaster recovery patterns: Ensure your static assets are available in multiple geographic locations using multi-region deployments with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]].

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for Amplify Hosting can be achieved through the following methods:

* Setting up [[billing]] alerts: Receive notifications when your monthly costs reach specific thresholds.
* Configuring usage-based deletion: Automatically delete older versions of files to reduce storage costs.
* Enforcing [[Budgets]]: Define [[Budgets]] for individual projects or teams to keep track of spending.

### Professional Exam Scenarios

#### Scenario 1

You are designing a global e-commerce platform using Amplify Hosting. Your primary requirement is low latency, and you want to minimize the number of requests between clients and the origin. Which pattern would you choose?

Correct answer: Deploy your Amplify Hosting app to multiple regions to serve static assets from the nearest edge location. Additionally, use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to distribute the content globally and implement [[Lambda@Edge]] to perform serverless transformations.

Incorrect answer: Store all static assets in a single [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and serve them directly from the bucket.

Justification: Serving assets directly from a single [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket does not take advantage of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]'s global network, resulting in higher latencies.

#### Scenario 2

Your organization has strict [[appsync|security]] requirements and wants to limit Amplify Console's ability to create and update [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] objects. How would you implement this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]?

Correct answer: Create a custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with limited permissions for Amplify Console.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-amplify-bucket*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:SourceVpce": "vpce-1234abcd"
        }
      }
    }
  ]
}
```

Incorrect answer: Grant Amplify Console full [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] permissions.

Justification: Full [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] permissions for Amplify Console may expose unnecessary actions and resources, potentially violating [[appsync|security]] requirements.