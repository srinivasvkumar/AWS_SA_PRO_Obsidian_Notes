Types of end points


While [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering enables you to privately connect VPCs, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] enables you to configure applications or services in VPCs as endpoints that your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connections can connect to. This is a more secure solution as there is less trust required.

Types of End points: 

- **Interface** - Create an _interface endpoint_ to send traffic to endpoint services that use a Network Load Balancer to distribute traffic. Traffic destined for the endpoint service is resolved using DNS.

- **GatewayLoadBalancer** - Create a _Gateway Load Balancer endpoint_ to send traffic to a fleet of virtual appliances using private IP addresses. You route traffic from your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to the Gateway Load Balancer endpoint using route tables. The Gateway Load Balancer distributes traffic to the virtual appliances and can scale with demand.

- **Gateway** - Create a _gateway endpoint_ to send traffic to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[dynamodb]] using private IP addresses. You route traffic from your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to the gateway endpoint using route tables. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Gateway endpoints]] do not enable [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]].
