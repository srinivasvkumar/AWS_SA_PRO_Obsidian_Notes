### Advanced Architecture

At its core, Copilot for Containers is a service that simplifies deployment, management, and scaling of containerized applications. It provides a unified UI for [[ecs]], [[eks]], and [[Fargate]] services. The architecture consists of several components:

- **Frontend**: A web application built using ReactJS and Redux. It communicates with the backend through RESTful APIs.
- **Backend**: Written in NodeJS, it uses AWS SDK to interact with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS Services]].
- **Task Planner**: Responsible for scheduling tasks across different cluster types ([[ecs]], [[eks]], [[Fargate]]) based on user input.
- **Data Store**: [[dynamodb]] is used for storing user configurations, task definitions, and metadata.

For global scale, Copilot uses [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to ensure low latency connections worldwide. This service directs user traffic to the nearest regional frontend server while ensuring high availability and automatic failover between multiple backend services.

### Comparison & Anti-Patterns