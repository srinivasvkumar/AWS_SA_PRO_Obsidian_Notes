### Advanced Architecture
At its core, Amazon Web Services' (AWS) [[ec2]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs) provide significant discounts on instance usage when purchased in advance. RIs offer three main benefits over On-Demand instances: capacity reservation, flexible payment options, and up to 75% discount compared to On-Demand pricing. The primary components of RI [[RDS_Instance_Types|internals]] include:
1. *Scope*: Regional or Zonal [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|reserved instances]].
2. *Instance Type*: Specific instance family and size.
3. *Term*: One-year or Three-year term.
4. *Purchase Options*: All Upfront, Partial Upfront, and No Upfront payments.

The underlying mechanics for Reservation Matching involve three phases:
1. *Exact Match*: If an exact match is found between your RI and running instance, it will receive a full benefit.
2. *Convertible Match*: Convertible RIs allow you to change the attributes (instance type, region, scope, tenancy) associated with your RI while retaining some portion of the original RI's purchase value.
3. *Linux/Windows unblended rates*: For any remaining compute usage that does not fit into either the Exact or Convertible matches, customers pay the Linux/Windows unblended rate, which is a blending of On-Demand, Reserved, and Spot prices.

### Comparison & Anti-Patterns
When considering using RIs, understanding their [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] and potential anti-patterns is essential. Here's a comparison table highlighting key differences between RIs and alternative services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] and On-Demand Instances:

| Feature                     | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]]    | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]       | On-Demand Instances   |
|-----------------------------|-----------------------|----------------------|-----------------------|
| Payment Terms              | Advance Purchase      | Auction-based        | Pay as you go          |
| Discount (% off On-Demand)  | Up to 75%             | Variable            | None                 |
| Instance Guarantee          | Yes                   | No                   | Yes                   |
| Capacity Guarantee         | Yes (Regional only)   | No                   | No                   |
| Use Case                    | Steady state workloads| Spike workloads      | Unpredictable workloads|

Common design mistakes include purchasing too many RIs at once without considering future growth requirements, misaligned instance types or regions, and insufficient attention to the financial implications.

### [[appsync|Security]] & Governance
Implementing complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and organization Service Control [[policies]] (SCPs) can help secure and govern your RIs effectively. Here's a JSON example demonstrating fine-grained control over instance usage within an account:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": [
            "t2.micro",
            "t3.micro"
          ]
        }
      }
    }
  ]
}
```
Cross-account access involves sharing RIs through ModifyChooseInstanceTypeOffering or CreateFleetRequests APIs. To enable this, ensure proper permissions are granted via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Additionally, enforce centralized management using [[organizations|AWS Organizations]] and restrict actions through Service Control [[policies]] (SCPs):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:Describe*",
        "ec2:ModifyInstanceAttribute",
        "ec2:CreateImage",
        "ec2:CreateSnapshot",
        "ec2:CancelConversionTask",
        "ec2:CreateTags",
        "ec2:CreateVolume*,
        "ec2:CreateVpnConnection",
        "ec2:Delete*",
        "ec2:Describe*",
        "ec2:Disable*",
        "ec2:Enable*",
        "ec2:Import*",
        "ec2:Monitor*",
        "ec2:MoveAddressToVpc",
        "ec2:RequestSpotInstances",
        "ec2:Reset*",
        "ec2:Start*",
        "ec2:Stop*",
        "ec2:UnassignPrivateIpAddresses",
        "ec2:
```