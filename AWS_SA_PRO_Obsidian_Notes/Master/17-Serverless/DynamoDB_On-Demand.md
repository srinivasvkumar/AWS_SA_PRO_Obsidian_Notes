Advanced Architecture
---------------------

At its core, [[dynamodb]] On-Demand provides automatic capacity management by enabling you to pay for only the read and write requests that your application actually performs. It eliminates the need for manual capacity planning, but it comes at a premium price compared to provisioned mode. The service scales elastically based on the demand, providing single-digit millisecond response times at any scale.

Internally, [[dynamodb]] On-Demand utilizes a combination of partitioned hash tables and B-trees to store data. This enables efficient handling of both key-based lookups and range queries. When using On-Demand, [[dynamodb]] automatically manages the underlying resources required to support your workload. To ensure consistent performance during heavy request rates, [[dynamodb]] uses a technique called "auto-scaling," which involves adding or removing capacity in small increments.

Global scale is achieved through [[dynamodb]]'s Global Tables feature. With Global Tables, you can replicate your [[dynamodb|DynamoDB tables]] across multiple regions to provide low-latency reads and writes. However, when using Global Tables with [[dynamodb]] On-Demand, all regions will consume capacity from the same pool, potentially affecting performance if one region experiences high traffic.

Comparison & Anti-Patterns
---------------------------

| Service               | When to Use                                                                      |
|-----------------------|-------------------------------------------------------------------------------|
| [[dynamodb]] On-Demand    | Unpredictable traffic patterns, infrequent usage, or new applications         |
| [[dynamodb]] Provisioned  | Predictable traffic patterns, known capacity requirements                     |
| [[dynamodb]] DAX          | High-performance [[api-gateway|caching]] layer for read-intensive workloads                   |
| [[dynamodb]] Streams     | Data processing, real-time analytics, and backup/restore                       |

Anti-patterns include:

* Using [[dynamodb]] On-Demand for predictable traffic patterns, resulting in unnecessary costs.
* Not monitoring [[dynamodb]] On-Demand usage, leading to unexpected charges.

[[appsync|Security]] & Governance
----------------------

To secure access to [[dynamodb]] On-Demand, follow these [[iam|best practices]]:

1. Implement strict [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for users, groups, and roles. Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:region:account-id:table/table-name"
      ]
    }
  ]
}
```

2. Enable encryption at rest using AWS Key Management Service ([[kms]]) keys.

3. Grant cross-account access via resource-based [[policies]]. Example resource-based policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::other-account-id:root"
      },
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:region:account-id:table/table-name"
    }
  ]
}
```

4. Implement Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] to restrict [[dynamodb]] usage.

Performance & Reliability
--------------------------

When working with [[dynamodb]] On-Demand, consider throttling limits and implement exponential backoff strategies. For example, if a request fails due to throttling, wait for a short period before retrying. Gradually increase the waiting time between retries until the request succeeds.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your [[dynamodb|DynamoDB tables]] across multiple regions using Global Tables. Ensure you have proper backup and restore mechanisms in place, such as point-in-time recovery and continuous backups.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls include monitoring and analyzing usage patterns, setting up [[billing]] alarms, and using AWS [[billing|Cost Explorer]] to visualize spending trends. Additionally, evaluate the potential savings from reserved capacity if your traffic patterns become more predictable.

Professional Exam Scenario
--------------------------

Scenario 1: A company has a web application that occasionally experiences spikes in traffic. They want to use [[dynamodb]] On-Demand to handle the variable load without worrying about capacity planning. What are the benefits and drawbacks of this approach?

Benefits:

* Eliminates capacity planning and manual adjustments.
* Automatically scales to meet demand.
* Simplifies infrastructure management.

Drawbacks:

* Higher cost compared to provisioned mode.
* Potential for performance issues during heavy traffic if not properly monitored.

Scenario 2: A media organization wants to store user preferences in a [[dynamodb]] table while ensuring low-latency access from different geographical locations. How would you design a solution using [[dynamodb]] On-Demand, and what trade-offs should be considered?

Solution:

* Create a [[dynamodb]] Global Table with replicas in each desired region.
* Utilize [[dynamodb]] On-Demand to eliminate capacity planning concerns.

Trade-offs:

* Increased cost compared to using provisioned mode.
* Possibility of decreased performance during peak traffic due to shared capacity across regions.