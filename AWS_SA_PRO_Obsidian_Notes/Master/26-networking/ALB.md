---
tags: [compute, load-balancing, security, service:alb, domain:compute-scaling-loadbalancing, concept:http, concept:tcp, concept:api-gateway]
---

1) An ALB only listens for HTTP and HTTPS traffic which uses the TCP protocol. It does not support UDP
2) ALBs do not provide REST APIs so you could not use an ALB to process API requests and forward them to the business logic function. that the functionality of [[api-gateway|API gateway]]

### Architecture Diagram
![[ALB vs NLB 2.png]]


### Architecture Diagram
![[ALB vs NLB 3.png]]


### Architecture Diagram
![[ALB vs NLB 1.png]]


### Architecture Diagram
![[ALB vs NLB.png]]
