---
tags: [compute, load-balancing, security, service:nlb, domain:compute-scaling-loadbalancing, concept:udp, concept:elastic-ip, concept:security-group]
---

1) Note that NLBs do not have [[appsync|security]] groups configured and pass connections straight to [[ec2]] instances with the source IP of the client preserved
2) NLB uses flow has algorithm to manage network traffic load balancing
3) The [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] supports the UDP protocol and can be placed in front of the application instance.
4) Unlike an [[ALB]] where ip's change frequently and nlb allows  you to assign elastic ip's per availability zone. This is the onlly navtive way to give clients a set of stable public IP's for whitelisting

### Architecture Diagram
![[ALB vs NLB 2.png]]


### Architecture Diagram
![[ALB vs NLB 3.png]]


### Architecture Diagram
![[ALB vs NLB 1.png]]


### Architecture Diagram
![[ALB vs NLB.png]]
