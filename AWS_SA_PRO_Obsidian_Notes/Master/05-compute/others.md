---
aliases:
- "AWS Outposts"
- "AWS Wavelength"
- "Other Compute Related Products"
---

# Other Compute Related Products

## AWS Outposts

- AWS Outposts is a fully managed service that extends AWS infrastructure, services, APIs, and tools to customer premises
- An Outpost is a pool of AWS compute and storage capacity deployed at a customer site
- AWS operates, monitors, and manages this capacity as part of an AWS Region
- We can create subnets on our Outpost and specify them when we create AWS resources such as [[ec2]] instances, [[ebs]] volumes, [[ecs]] clusters, and [[rds]] instances
- Instances in Outpost subnets communicate with other instances in the AWS Region using private IP addresses, all within the same [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
- Not every AWS service is supported within an Outpost, for a list see: https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html#services

## AWS Wavelength

- AWS [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]] is just like having an Availability Zone in a phone carrier's 'edge' network
- [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]] deploys standard AWS compute and storage services to the edge of telecommunication carriers' 5G networks
- We can extend a virtual private cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]) to one or more [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]] Zones
- We can then use AWS resources such as Amazon Elastic Compute Cloud (Amazon [[ec2]]) instances to run the applications that require low latency or edge resiliency within the [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]] Zone
- AWS resources on [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]]:
    - [[ec2]] Auto Scaling
    - [[eks]] clusters
    - [[ecs]] clusters
    - [[ec2]] System Manager
    - [[cloudwatch]], [[cloudtrail]]
    - [[cloudformation]]
    - Application Load Balancers
- The services in [[AWS_SA_PRO_Obsidian_Notes/Master/Edge_Services/Wavelength|Wavelength]] are part of a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] that is connected over a reliable connection to an AWS region