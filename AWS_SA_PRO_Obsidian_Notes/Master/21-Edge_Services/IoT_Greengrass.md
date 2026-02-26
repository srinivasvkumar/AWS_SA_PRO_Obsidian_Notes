**Advanced Architecture of [[iot]] Greengrass**

[[iot]] Greengrass is a service that extends AWS to edge devices, allowing them to run [[lambda|AWS Lambda]] functions, keep device data in sync, and communicate with each other securely. The following are advanced architecture components:

* **Deployment Units**: They consist of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], their dependencies, and resources required by the [[lambda]] function. Deployment units can be uploaded as a package or deployed using container images.
* **Core Device**: A core device runs the Greengrass core software and manages deployment units, local resources, and communication between edge devices and the cloud.
* **Group**: A group is a collection of devices connected to a core device. Groups can contain both [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and other groups.
* **Global Scaling**: [[iot]] Greengrass supports global scaling through centralized management of cores and groups. Cores connect to the cloud via the [[iot]] Greengrass [[api-gateway|API Gateway]], enabling automatic updates and deployments.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[iot]] Core | Large fleets of remote devices requiring publish/subscribe messaging and low-power connectivity options |
| [[iot]] Analytics | Real-time processing and enrichment of [[iot]] data, followed by storage in long-term storage like [[Srinivas_Notes/S3|S3]] |
| [[iot]] Greengrass | Local compute, machine learning, and data [[api-gateway|caching]] for [[iot]] applications |

Anti-patterns include using [[iot]] Greengrass without considering throttling limits or not implementing proper [[appsync|security]] measures.

**[[appsync|Security]] & Governance**

[[iot]] Greengrass uses fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for controlling access to AWS services. Example JSON policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iot:Publish",
                "iot:Receive"
            ],
            "Resource": [
                "arn:aws:iot:*:*:topic/*"
            ]
        }
    ]
}
```

Cross-account access requires creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account, granting permissions, then assuming the role from the destination account. Organization SCPs can enforce restrictions on [[iot]] Greengrass usage across accounts.

**Performance & Reliability**

Throttling limits apply to various [[iot]] Greengrass operations. For example, there is a limit of 10,000 simultaneous connections per Greengrass group. To handle throttling, implement exponential backoff strategies, ensuring reliable message delivery.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve configuring multiple Greengrass cores in different regions or availability zones. In case of failure, edge devices will automatically switch to the next available core.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring resource usage and deleting unused cores and groups. Calculation examples:

* Cost of running a Greengrass core: ($0.10 per hour \* number of hours) + ($0.04 per GB per month \* GB of data stored)
* Cost of sending messages: ($0.000000417 per MB \* number of MB sent)

**Professional Exam Scenarios**

Scenario 1: An organization has smart cameras installed at its manufacturing facilities for real-time video analysis. They want to reduce latency and maintain privacy by processing videos locally. Propose a solution using [[iot]] Greengrass.

Correct answer: Implement [[iot]] Greengrass at the edge to process video frames, perform real-time analysis using [[lambda|AWS Lambda]] functions, and transmit metadata to the cloud for further processing. This reduces latency while maintaining privacy.

Incorrect answer: Using [[iot]] Core instead of Greengrass would result in higher latency due to the need to send raw video data to the cloud.

Scenario 2: A company wants to synchronize data between [[iot]] devices and [[iot|AWS IoT Core]] but needs to minimize data transfer costs. What approach should they take?

Correct answer: Configure [[iot]] Greengrass to filter and aggregate data before transmitting it to [[iot]] Core. This minimizes data transfer costs and improves overall system performance.

Incorrect answer: Implementing [[iot]] Core without Greengrass could lead to excessive data transfer costs and potential network congestion.