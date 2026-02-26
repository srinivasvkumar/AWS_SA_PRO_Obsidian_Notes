**[[RDS_Instance_Types|1. Advanced Architecture]]**

RoboMaker is a fully managed service that makes it easy to develop, test, and deploy intelligent robotics applications at scale. It provides an integrated set of tools and services that simplify building robotics software. The underlying architecture consists of the following components:

- **Simulation Service**: A highly scalable, distributed simulation system for running simulations in parallel across multiple machines.
- **Fleet Management Service**: A service for managing fleets of robots, including deployment, remote procedure calls, and configuration management.
- **Connectors**: Pre-built connectors to popular robotics middleware such as ROS, Gazebo, and Unity.

To build globally scalable systems with RoboMaker, you can create multi-region deployments using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or by setting up separate RoboMaker resources in each region. This allows you to reduce latency for users worldwide and ensures high availability.

When designing RoboMaker solutions, understanding its internal mechanics is crucial. RoboMaker leverages [[aws-batch|AWS Batch]] to run simulations in the background. Each simulation job runs on an Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Elastic Container Service (ECS)]] cluster configured with container instances based on the desired machine type. To ensure efficient resource utilization, RoboMaker uses AWS [[Fargate]] to manage containers without provisioning or managing servers.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service              | Use Case                                                          |
| -------------------- | ------------------------------------------------------------------ |
| RoboMaker            | Intelligent robotics application development, testing, and deployment |
| [[Lambda@Edge]]         | Lightweight edge computing tasks, e.g., image resizing             |
| [[iot|AWS IoT Core]]        | [[iot]] device connectivity, messaging, and rules engine               |
| [[iot|AWS Greengrass]]      | Local compute, messaging, data [[api-gateway|caching]], and ML inference           |

Anti-patterns include using RoboMaker for simple [[iot]] tasks better suited for [[iot|AWS IoT Core]] or [[Lambda@Edge]]. Overcomplicating edge computing scenarios with RoboMaker instead of using [[iot|AWS IoT]] Greengrass is another common mistake.

**[[RDS_Instance_Types|3. Security & Governance]]**

For cross-account access, you need to set up an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the RoboMaker account allowing required actions. For example, the following JSON policy grants permission for creating simulation jobs from another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "robomaker:StartSimulationJob"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {