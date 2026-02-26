---
aliases:
- "Regional and Global AWS Architecture"
---

# Regional and Global AWS Architecture

- There are 3 main type of architectures:
    - Small scale architectures: one region/one country
    - Small architecture with [[dr]]: one region + backup region for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]
    - Multiple region based systems
- Architectural components at global level:
    - Global Service Location and Discovery
    - Content Delivery (CDN) and optimization
    - Global [[route53|health checks]] and Failover
- Regional components:
    - Regional entry point
    - Scaling and resilience
    - Application services and components
![Regional and Global Architecture](AWS_SA_PRO_Obsidian_Notes/Master/05-compute/images/RegionalandGlobalInfrastructure2.png)


Web Tier : Customer facing layer. There would be regional based services like [[ALB]] or [[api-gateway|API gateway]] dependig on application architecture. Abstracts customers from underlying architecture.
Compute Tier: The infra to web tier is provided by compute tier using [[ec2]], [[lambda]] or containers.
Storage Services: Service like [[ebs]], [[efs]] or [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]].
Data Storage: Products like [[rds]], [[aurora]], [[dynamodb]], and [[redshift]].
[[api-gateway|Caching]]: Elastic cache for general [[api-gateway|caching]], [[dynamodb]] acclerator(DAX)
App Services: [[kinesis]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Step Functions]], [[sqs]], [[sns]]



### Architecture Diagram
![[Regional and Global AWS Architecture.png]]
