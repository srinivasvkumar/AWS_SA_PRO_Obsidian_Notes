
---
tags: [database, scalability, service:dynamodb, domain:databases, concept:global-tables, concept:ttl, concept:on-demand-scaling, concept:provisioned-scaling]
---

1) The [[dynamodb|DynamoDB global tables]] solution is the only database solution that will allow writes in both AWS Regions
2) Amazon [[dynamodb]] Time to Live (TTL) allows you to define a per-item timestamp to determine when an item is no longer needed. Shortly after the date and time of the specified timestamp, [[dynamodb]] deletes the item from your table without consuming any write throughput. TTL is provided at no extra cost as a means to reduce stored data volumes by retaining only the items that remain current for your workload’s needs.
3) With On-Demand Scaling you don't need to think about provisioning throughput. Instead, your table will scale all behind the scenes automatically. Extreme spikes in load can occur and be handled seamlessly by AWS. This also brings a change in the cost structure for [[dynamodb]]. With On-Demand Scaling, instead of paying for provisioned throughput, you pay per read or write request.
4) Remember for Provisioned with Auto-Scaling you are basically paying for throughput 24/7. Whereas for On-Demand Scaling you pay per request. This means for applications still in development or low traffic applications, it might be more economical to use On-Demand Scaling and not worry about provisioning throughput. However, at scale, this can quickly shift once you have a more consistent usage pattern

![[Screenshot 2026-02-12 at 10.25.26 PM.png]]



### AI Linked Diagram
![[Screenshot 2026-01-03 at 8.34.42 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-01 at 11.45.02 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-03 at 8.53.14 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-01 at 11.32.10 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-03 at 8.25.38 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-03 at 10.10.30 PM.png]]
