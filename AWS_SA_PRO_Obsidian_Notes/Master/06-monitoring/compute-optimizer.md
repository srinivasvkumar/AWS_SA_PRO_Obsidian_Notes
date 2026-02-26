---
aliases:
- "AWS Compute Optimizer"
---

# [[AWS Compute Optimizer]]

- [[AWS Compute Optimizer]] is a service that analyzes our AWS resources' configuration and utilization metrics to provide us with rightsizing recommendations
- Generates optimization recommendations to reduce the cost and improve the performance of your workloads
- Supported resources and requirements:
    - [[ec2]] instances
    - Auto Scaling Groups
    - [[ebs]] volumes
    - [[lambda]] functions
    - [[ecs]] and [[ecs]] [[Fargate]]
    - Commercial Software Licenses
    - [[rds]] DB instances and storage
- Compute Optimizer is opt-in
- Analyzing metrics: after opt in, Compute Optimizer begins analyzing the specifications and the utilization metrics of resources from Amazon [[cloudwatch]] for the last 14 days
- Enhancing recommendations: we can enhance recommendations by activating recommendation preferences, such as the enhanced infrastructure metrics paid feature