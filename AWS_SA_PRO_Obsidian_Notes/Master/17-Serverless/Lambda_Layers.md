**[[RDS_Instance_Types|1. Advanced Architecture]]**

At its core, [[lambda|Lambda Layers]] is a service that allows you to centrally manage common code and dependencies within your serverless applications. It simplifies code reuse and reduces overhead by enabling developers to package and deploy libraries, runtime executables, or other assets separately from their function code.

Internally, [[lambda|Lambda Layers]] consist of two primary components:

- **Layer Versions**: Immutable snapshots containing the contents of a layer at creation time. They can be managed within an account or shared across accounts.
- **Layer Aliases**: Point to specific layer versions, allowing easy version management and control.

When it comes to [[RDS_Instance_Types|global scale considerations]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] using layers will automatically run in the same region as the layer. However, if a function is invoked in a different region than its associated layer, AWS will handle the necessary data transfers behind the scenes. This introduces additional latency due to cross-region communication.

The 'Under the Hood' mechanics involve several [[kms|key concepts]]:

- Each layer version has a unique ARN, which includes the region and AWS account ID.
- The layer's contents are extracted and added to the function's deployment package during invocation.
- Multiple layers can be used within a single function, but there's a maximum limit of five layers per function.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service               | Use Cases                                                              | Anti-Patterns                                                   |
| --------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------- |
| [[lambda|Lambda Layers]]        | Code reusability, dependency management                              | Overusing layers, adding unnecessary complexity             |
| [[lambda]] Data Durable  | Infrequent read/write operations, long-term storage                  | High frequency read/write operations, real-time processing      |
| [[lambda]] Extensions    | Custom runtime logic, performance profiling, monitoring              | Tightly coupling application logic with extensions              |
| ECR + [[Fargate]]         | Dockerized workloads, custom runtimes, long-running processes          | Short-lived stateless tasks, event-driven architecture           |

Common design mistakes include overusing layers, leading to increased complexity and potential [[appsync|security]] risks. Another anti-pattern is storing sensitive data within layers, which should be avoided.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve granting permissions to specific layer versions or aliases. For example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "lambda:GetLayerVersion",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:layer:LAYER_NAME:VERSION"
    }
  ]
}
```

Cross-account access can be configured using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, allowing functions in one account to access layers in another. Additionally, Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] can enforce restrictions on layer usage across member accounts.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits for [[lambda|Lambda Layers]] depend on the number of functions using them. A single layer version can support up to 500 functions. If exceeded, throttling occurs, causing [[api-gateway|errors]] when attempting to add more functions. In such cases, exponential backoff strategies should be implemented.

HA/DR patterns can leverage multiple layers in different regions, ensuring high availability and redundancy. However, this increases costs and adds complexity.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved by applying tags to layers, allowing [[organizations]] to track spending and set budget alerts. Calculation examples:

- Basic layer: $0, assuming no data transfer or storage costs.
- Large layer (1GB): ~$0.10/month (assuming 100 PUT/GET requests and 1 GB data transfer).

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You are designing a serverless application that requires frequent updates to shared libraries. Which solution would best meet these requirements while minimizing maintenance efforts?

Correct Answer: [[lambda|Lambda Layers]]
Justification: By using [[lambda|Lambda Layers]], you can centralize library management and reduce maintenance overhead. Updating a library only requires updating the corresponding layer, which then propagates to all functions using it.

Incorrect Answers:

- [[lambda]] Data Durable: While this service can store data, it isn't designed for managing libraries or dependencies.
- [[lambda]] Extensions: These are meant for custom runtime logic, performance profiling, and monitoring rather than library management.
- ECR + [[Fargate]]: Although capable of handling Dockerized workloads, they don't offer the same level of simplicity and integration as [[lambda|Lambda Layers]].

Scenario 2:
Your organization wants to restrict specific teams from creating or modifying [[lambda|Lambda Layers]]. How can you achieve this using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs)?

Correct Answer:
Create an [[SCP]] that denies relevant actions (e.g., `lambda:CreateLayerVersion`, `lambda:UpdateLayerVersion`) for the desired teams or organizational units (OUs).

Incorrect Answers:

- Implementing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] directly on the layers: While possible, this approach doesn't scale well and may lead to inconsistent configurations.
- Using tag-based [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]: Tagging layers alone won't prevent unauthorized actions; you still need to define appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].