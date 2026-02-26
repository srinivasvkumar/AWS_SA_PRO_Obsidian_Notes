## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] is a small, ruggedized, and secure edge computing and data transfer device provided by AWS. It is designed to operate in harsh environments where traditional servers or data centers cannot reach. The device has internal storage for data, runs [[iot|AWS IoT]] Greengrass, and can execute [[lambda|AWS Lambda]] functions.

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] uses an ARM-based processor, which provides sufficient power for running [[iot]] Greengrass and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. Each device comes with two 4 GB EMMC drives for local storage, providing up to 14 TB of usable capacity after compression and deduplication. Devices connect to each other via WiFi or Ethernet, forming a local network that allows them to communicate and synchronize data.

Global scale is achieved through the use of [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]], which enables customers to leverage the same infrastructure, services, APIs, and tools across their on-premises and cloud resources. [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices integrate seamlessly with Outposts, allowing customers to extend their AWS environment to remote locations without having to worry about managing additional hardware.

## Comparison & Anti-Patterns

| Service            | Use Case                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------- |
| [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]]          | Edge computing, disconnected operations, remote locations, temporary work sites           |
| [[snow|Snowball Edge]]     | Large-scale data transfers, local processing, long-term data storage                       |
| [[Storage Gateway]]   | On-premises file sharing, backup, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], and tiered storage management         |
| [[Collab_Notes_detailed/Migration_and_Transfer/DataSync|DataSync]]          | Automated, high-performance data transfer between on-premises storage and AWS Storage |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] as a primary data center replacement or for large-scale data processing, as it is not designed for these purposes. Another mistake is attempting to manage [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices directly instead of leveraging [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]] or [[snow|Snow Family]] Management software.

## [[appsync|Security]] & Governance

To ensure [[appsync|security]] and governance when using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]], you must follow [[iam|best practices]] such as enforcing encryption at rest and in transit, applying least privilege access principles, and monitoring device activity using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] and [[cloudwatch]].

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve granting cross-account access for specific actions, like `s3:PutObject` or `lambda:InvokeFunction`. Here's an example JSON policy snippet:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket/*"
            ],
            "Principal": {
                "AWS": [
                    "arn:aws:iam::123456789012:role/MySnowconeRole"
                ]
            }
        },
        {
            "Effect": "Allow",
            "Action": [
              "lambda:InvokeFunction"
            ],
            "Resource": [
                "arn:aws:lambda:us-east-1:123456789012:function:MyLambdaFunction"
            ],
            "Principal": {
                "AWS": [
                    "arn:aws:iam::123456789012:role/MySnowconeRole"
                ]
            }
        }
    ]
}
```
Organization Service Control [[policies]] (SCPs) should be used to enforce [[control-tower|guardrails]] and centralize control over multiple accounts. For instance, you could create an [[SCP]] to restrict [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices from creating new [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets or [[ec2]] instances.

## Performance & Reliability

When working with [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]], you need to consider throttling limits and implement appropriate backoff strategies. For example, if you exceed the maximum number of API calls per minute, you might experience [[api-gateway|errors]] or connection timeouts. To mitigate these issues, implement exponential backoff strategies using SDKs or client libraries.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns depend on your specific use case. For edge computing scenarios, you can deploy redundant [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices and configure automatic failover using [[iot|AWS IoT]] Greengrass. In cases where data durability is critical, enable versioning and lifecycle [[policies]] on associated [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] involve monitoring usage metrics and setting alarms based on specific thresholds. This can help you detect unexpected costs early and take action before they become significant. Additionally, you can optimize costs by enabling data compression and deduplication features within [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices.

For example, let's say you have a [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] device generating 10 TB of data every month. Assuming a data transfer cost of $0.09/GB, the monthly cost would be around $900. By enabling data compression and deduplication, you could reduce the amount of data transferred by 50%, resulting in a reduced monthly cost of approximately $450.

## Professional Exam Scenarios

### Scenario 1:

Suppose you're tasked with implementing a solution to collect sensor data from remote oil [[eb|platforms]] and transmit it to AWS for real-time analysis. The platform has limited network connectivity, and data privacy is essential. Which AWS services would you choose?

Correct answer: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]], [[iot|AWS IoT Core]], [[iot|AWS IoT Analytics]], and [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]].

Explanation: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices can be deployed on the oil [[eb|platforms]] to collect sensor data and run [[iot|AWS IoT]] Greengrass for edge computing. [[iot]] Core can be used to securely transmit data to the cloud, while [[iot]] Analytics processes and analyzes the data. Finally, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] ensures private connectivity between the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] devices and [[iot]] Core, ensuring data privacy.

Incorrect answers: [[snow|Snowball Edge]], [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]], or AWS [[Storage Gateway]] because these services are not suitable for remote locations with limited network connectivity.

### Scenario 2:

You need to migrate a large dataset stored on-premises to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Due to regulatory requirements, you cannot transmit the data over the internet. What AWS service would you use?

Correct answer: [[Snowball]] or [[Snowmobile]].

Explanation: Since data transfer over the internet is not allowed, [[Snowball]] and [[Snowmobile]] provide offline data migration options. [[Snowball]] is an edge computing device similar to [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]] but designed for large-scale data migrations. [[Snowmobile]] is a truck-sized data transport solution for even larger datasets. Both services support importing data into Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

Incorrect answers: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowcone|Snowcone]], [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]], or AWS [[Storage Gateway]] because these services do not offer offline data migration capabilities.