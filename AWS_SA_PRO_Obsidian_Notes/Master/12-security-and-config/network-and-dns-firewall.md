---
aliases:
- "AWS Network Firewall"
- "AWS Network and DNS Firewalls"
- "Route 53 Resolver DNS Firewall"
---

# AWS Network and DNS Firewalls

## AWS Network Firewall

- AWS Network Firewall is a stateful, managed, network firewall and intrusion detection and prevention service for your virtual private cloud
- It is a managed network appliance used to filter incoming/outgoing traffic
- In a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] it is deployed in a separate subnet with a Firewall Endpoint
- Has a flexible rules engine which gives a fine-grained control over network traffic
- Resources send traffic to the firewall subnet, the firewall will redirect traffic to the gateway (internet gateway, [[Transit gateway]], DX connection, IPSEC [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]])
- The firewall endpoint uses gateway load balancer
- Route table configuration is needed for the protected subnet to be able to send outgoing traffic to the gateway load balancer
- Network Firewall deployment can be managed with AWS [[organizations]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Firewall Manager]]
- Network Firewall features:
    - Stateful and stateless firewall
    - Intrusion Prevention System (IPS)
    - Web filtering
- In the firewall subnet we should not deploy any resources
- For HA we should allocate a subnet per AZ

## Route 53 Resolver DNS Firewall

- Allows us to filter and regulate outbound DNS traffic for VPCs
- Requests route through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver for DNS queries, the DNS Firewall will help prevent DNS exfiltration of data
- We can monitor and control the domains application can query
- We can use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Firewall Manager]] to centrally configure and manage DNS Firewall