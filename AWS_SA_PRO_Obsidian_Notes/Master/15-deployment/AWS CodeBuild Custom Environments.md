```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[AWS CodeBuild]] Custom Environments

#### UNDER-THE-HOOD MECHANICS:
[[AWS CodeBuild]] allows you to create custom build environments, which are essentially Docker images that contain your build tools and dependencies. These environments are hosted within the CodeBuild service, where they execute build jobs securely in isolated instances.

**Internal Data Flow:**
1. **Image Pull**: When a build job is initiated, [[AWS CodeBuild]] pulls the specified Docker image from Amazon ECR or another supported registry.
2. **Environment Setup**: The environment container is initialized with the necessary resources and permissions.
3. **Execution Phase**: Build commands are executed within this container. Logs and artifacts generated during execution are stored.
4. **Cleanup**: After completion, the container is disposed of to free up resources.

**Consistency Models:**
- **Stateless Execution**: Each build job starts fresh with a new instance of the Docker image, ensuring statelessness between builds.

#### EXHAUSTIVE FEATURE LIST:
1. **Custom Images**: Use custom Docker images from [[Amazon ECR]] or [[Docker Hub]].
2. **Environment Variables**: Set environment variables for build jobs.
3. **Build Commands**: Specify detailed commands to be executed during the build process.
4. **Privileged Mode**: Run containers with elevated privileges if required.
5. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Integration**: Deploy builds within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] for enhanced [[appsync|security]] and network isolation.
6. **Logs & Artifacts**: Detailed logs and artifacts management.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Create Custom Docker Image**:
    - Build your custom image containing build tools and dependencies.
    - Push the image to [[Amazon ECR]] or another supported registry.
2. **Define CodeBuild Project**:
    - Specify the image URI in the project configuration.
3. **Configure Environment Variables**: Set necessary variables for the build process.
4. **Add Build Commands**: Define specific commands for the build lifecycle (pre-build, build, post-build).
5. **Deploy and Test**: Execute a test build to ensure everything works as expected.

#### TECHNICAL LIMITS:
- Maximum size of Docker image: 10 GB
- Number of concurrent builds per [[AWS]] account: 25 by default (can be increased with a request)
- Disk storage for the build environment: 50 GB

**References**: 
- [AWS CodeBuild Quotas](https://docs.aws.amazon.com/codebuild/latest/userguide/limits.html)

#### CLI & API GROUNDING:
1. **Create a Project using Custom Image**:
    ```bash
    aws codebuild create-project --name MyCustomProject \
    --source-version master \
    --artifacts '{\"type\": \"NO_ARTIFACTS\"}' \
    --environment '{\"computeType\": \"BUILD_GENERAL1_SMALL\", \"image\": \"my-custom-image:latest\", \"privilegedMode\": false, \"type\": \"LINUX_CONTAINER\"}' \
    --service-role arn:aws:iam::123456789012:role/CodeBuildServiceRole
    ```
2. **Start a Build**:
    ```bash
    aws codebuild start-build --project-name MyCustomProject
    ```

#### PRICING & TCO:
- Pay-as-you-go model based on compute time.
- Storage and data transfer costs apply for artifacts stored in [[Amazon S3]].

**Optimization Strategies**:
1. **Use [[00_Master_Links_and_Intro|Spot Instances]]**: Reduce cost by using [[00_Master_Links_and_Intro|spot instances]] where possible.
2. **Minimize Build Time**: Optimize Docker images to reduce build times.
3. **Clean Artifacts**: Regularly clean up old build artifacts to minimize storage costs.

#### COMPETITOR COMPARISON:
- [[Jenkins]]: Offers more flexibility but requires more setup and maintenance.
- [[GitHub Actions]]: Easier integration with GitHub repositories, simpler configuration for small projects.
- [[CircleCI]]: Rich feature set including [[api-gateway|caching]] mechanisms, optimized for [[cicd|CI/CD]] pipelines.

**Architectural Trade-offs**:
- [[cicd|AWS CodeBuild]] offers managed environments and built-in [[appsync|security]] features, but requires setting up custom images for specific needs.

#### INTEGRATION:
1. **[[cloudwatch|CloudWatch Logs]]**: Automatically integrated for [[vpc-flow-logs|logging]] build activities.
2. **[[00_Master_Links_and_Intro|IAM]] Roles**: Use [[00_Master_Links_and_Intro|IAM]] roles to manage permissions for accessing resources like [[Amazon S3]] buckets or [[Amazon ECR]] repositories.
3. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Configuration**: Build jobs can be executed within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] for network isolation and [[appsync|security]].

#### USE CASES:
1. **Enterprise Continuous Integration/Continuous Deployment ([[cicd|CI/CD]])**: Automate build, test, and deployment processes.
2. **Custom Build Environments**: Support complex builds that require specific tools or dependencies.
3. **Multi-Region Builds**: Leverage [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] configurations for secure cross-region builds.

#### DIAGRAMS:
1. **CodeBuild Custom Environment Architecture** (Placeholder)
   ```
    +---------------------+       +--------------+
    | Docker Image        |  -->  | CodeBuild    |
    | (ECR/Docker Hub)    |       | Instance     |
    +---------------------+       +--------------+
                                    ^             |
                                    |             v
                               +----------+  +---------+
                               | Artifacts|  | Logs    |
                               +----------+  +---------+
   ```
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Integration** (Placeholder)
   ```
    +------------+       +-------------+      +-----------------+
    | Build      |<----->| CodeBuild VPC|----->| Other Resources |
    | Environment|       +-------------+      | (S3, ECR)        |
    +------------+                               +-----------------+
   ```

#### EXAM TRAPS:
1. **Misunderstanding Image Size Limits**: Not realizing the 10 GB limit for Docker images.
2. **Privileged Mode Misuse**: Incorrectly assuming all builds require privileged mode.

#### DECISION MATRIX:
| Feature                | Use Case                      |
|------------------------|-------------------------------|
| Custom Images          | Complex build requirements    |
| Environment Variables  | Flexible configuration        |
| Build Commands         | Detailed control over process |
| [[Master/Srinivas_Notes/VPC|VPC]] Integration        | Secure network isolation      |

#### 2026 UPDATES:
- Enhanced support for newer Docker versions and container orchestration.
- Improved integration with AWS [[appsync|security]] services.

#### EXAM SCENARIOS:
1. **Configuring Custom Image**: Given a scenario, configure [[CodeBuild]] to use a custom image from ECR.
2. **Troubleshooting Builds**: Identify common issues in build configurations and resolve them.

#### INTERACTION PATTERNS:
- **Service-to-Service Interactions**:
    - CodeBuild interacts with [[iam]] for role-based access control.
    - CodeBuild integrates with [[cloudwatch]] for [[vpc-flow-logs|logging]] purposes.

#### STRATEGIC ALIGNMENT:
Each feature is aligned to support the [[cicd|CI/CD]] pipeline, focusing on automation and [[appsync|security]] in build processes.

#### PROFESSIONAL TONE & AUDIENCE:
Content tailored specifically for SAP-C02 candidates, providing high-value technical details and exam preparation insights.

#### PRIORITIZATION & CLARITY:
Focus on [[kms|key concepts]] essential for passing the exam while ensuring explanations are clear and concise.

#### GROUNDING & VALIDATION:
References to official AWS documentation and whitepapers ensure accuracy and alignment with the latest AWS landscape.

### Audit for [[AWS CodeBuild]] Custom Environments

#### Overview:
[[AWS CodeBuild]] allows you to build, test, and deploy your applications using custom environments. A custom environment gives developers the flexibility to define their own build process, including the operating system, runtime, and any necessary tools or dependencies.

#### Enhancement Requirements:

1. **Distractor Analysis:**
   - **Scenario 1:** If the requirement is to manage [[cicd|CI/CD]] pipelines for containerized applications using managed services, CodeBuild might not be the best choice since [[AWS CodePipeline]] offers more seamless integration with [[00_Master_Links_and_Intro|other AWS services]] like [[ecs]] and [[eks]].
   - **Scenario 2:** For simple build scenarios where a predefined environment suffices, selecting a custom environment in CodeBuild would add unnecessary complexity and overhead.
   - **Scenario 3:** When dealing with large-scale continuous delivery requiring advanced feature sets such as parallelism or multi-region deployments, [[AWS Amplify]] or GitHub Actions might be more suitable due to their built-in support for these scenarios.
   - **Scenario 4:** If the organization primarily uses GitLab for version control and [[cicd|CI/CD]] processes, using CodeBuild would require additional integration steps rather than leveraging GitLab's native [[cicd|CI/CD]] pipelines.
   - **Scenario 5:** For [[organizations]] focused on serverless application deployment, [[AWS SAM (Serverless Application Model)]] provides a more streamlined experience compared to the custom environment setup in CodeBuild.

2. **The "Most Correct" Logic:**
   - **Performance vs. Cost Trade-offs:** Custom environments provide maximum flexibility and control over the build process. However, this comes at the cost of increased management overhead (e.g., setting up the environment from scratch). Managed runtime environments provided by [[AWS CodeBuild]] are more cost-effective but offer less customization.
   - **Scalability:** Custom environments can scale better for complex builds that require specific configurations or third-party software dependencies. This flexibility is crucial in enterprise settings where standard environments may not meet all build requirements.

3. **Enterprise Governance:**
   - **[[organizations|AWS Organizations]]:** Utilize [[AWS Organizations]] to manage CodeBuild resources across multiple accounts and regions, ensuring consistent governance [[policies]].
   - **Service Control [[policies]] (SCPs):** Implement SCPs to restrict access and control over which users or roles can create or modify CodeBuild projects and environments. 
   - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Share CodeBuild custom environment configurations securely between AWS accounts within an organization to streamline resource management and reduce duplication.
   - **[[00_Master_Links_and_Intro|CloudTrail]]:** Enable [[00_Master_Links_and_Intro|CloudTrail]] [[vpc-flow-logs|logging]] for auditing all changes made in [[CodeBuild]], ensuring compliance with internal [[policies]] and external regulations.

4. **Tier-3 Troubleshooting:**
   - **Complex Failure Modes:** Custom environments can encounter issues like misconfigured [[kms]] key [[policies]] leading to access denials during the build process.
     - **Example Scenario:** If a custom environment requires encrypted artifacts or secrets, incorrect permissions in the [[kms]] key policy will prevent CodeBuild from decrypting these resources. This results in build failures.
   - **Resolution Steps:**
     1. Verify that the [[00_Master_Links_and_Intro|IAM]] role associated with the [[CodeBuild]] project has the necessary permissions to access the specified [[kms]] key.
     2. Check the [[kms]] key [[policies]] and ensure they allow decryption actions by the CodeBuild service.

5. **Decision Matrix (Use Case vs. Solution):**

| Use Case                             | Recommended Solution                |
|--------------------------------------|-------------------------------------|
| Simple [[cicd|CI/CD]] with predefined envs    | [[AWS CodePipeline]]               |
| Customized build environments        | [[cicd|AWS CodeBuild]] - Custom Environment  |
| Containerized application [[cicd|CI/CD]]      | [[AWS ECS]] + [[AWS EKS]]          |
| Serverless Application Deployment    | [[AWS SAM]]                        |
| GitLab-based [[cicd|CI/CD]] processes         | GitLab [[cicd|CI/CD]]                        |

6. **Recent Updates (2026):**
   - **GA Features:** Look out for any newly General Availability (GA) features in CodeBuild related to enhanced [[appsync|security]], more granular control over environment customization, or improved integration with [[00_Master_Links_and_Intro|other AWS services]].
   - **Deprecated Items:** Stay informed about deprecation notices for older [[CodeBuild]] features that may affect current configurations and plan for necessary updates.

### Conclusion:
By thoroughly analyzing distractors, evaluating trade-offs, integrating enterprise governance practices, addressing complex troubleshooting scenarios, creating a decision matrix, and staying updated with recent developments, this audit provides a comprehensive [[nonportable_instructions|review]] of AWS CodeBuild Custom Environments.