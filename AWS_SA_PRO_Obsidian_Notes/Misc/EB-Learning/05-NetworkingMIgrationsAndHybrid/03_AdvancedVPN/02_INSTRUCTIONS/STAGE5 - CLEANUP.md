---
aliases:
- "Advanced Highly-Available Dynamic Site-to-Site VPN"
---

# [[STAGE4 - BGP ROUTING AND TESTING|Advanced Highly-Available Dynamic Site-to-Site VPN]]

# STAGE 5 - CLEANUP

Return to https://console.aws.amazon.com/vpc/home?region=us-east-1#VpnConnections:sort=VpnConnectionId

Delete both [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Connections

Move to https://console.aws.amazon.com/vpc/home?region=us-east-1#CustomerGateways:sort=CustomerGatewayId

Delete both Customer Gateways you created earlier

Open a new TAB https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/

- Delete the ONPREM stack

** Wait for the [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections to full delete **

- Delete the AWS Stack


The Account is now back in the same state as when you started
Thanks for using https://learn.cantrill.io 