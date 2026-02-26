When a Fargate task is launched, its elastic network interface requires a route to the internet to pull container images. If you receive an error similar to the following when launching a task, it is because a route to the internet does not exist:

CannotPullContainerError: API error (500): Get https://111122223333.dkr.ecr.us-east-1.amazonaws.com/v2/: net/http: request canceled while waiting for connection"

To resolve this issue, you can:

- For tasks in public subnets, specify **ENABLED** for **Auto-assign public IP** when launching the task.

- For tasks in private subnets, specify **DISABLED** for **Auto-assign public IP** when launching the task, and configure a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT gateway]] in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to route requests to the internet.