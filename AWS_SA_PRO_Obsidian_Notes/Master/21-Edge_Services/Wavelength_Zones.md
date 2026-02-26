## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones provide single-digit millisecond latency by embedding AWS infrastructure within the telco's edge network. This is achieved through a combination of [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]], Local Zones, and 5G/4G networks. The following diagram illustrates [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zone [[RDS_Instance_Types|internals]]:

```mermaid
graph LR
    subgraph Mobile Network Operator (MNO)
        teleco_edge[Telco Edge]
    end
    subgraph AWS Infrastructure
        outpost[Outpost]
        wz_control_plane[Wavelength Control Plane]
    end
    subgraph Customer VPC
        vpc[VPC]
        subnet[Subnet]
        app[Application]
    end
    
    teleco_edge -- Connectivity --> outpost
    outpost -- Management Plane --> wz_control_plane
    vpc -- Peering Connection --> outpost
    app -- Running in --> subnet
```

To achieve global scale, [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones rely on MNOs to host AWS infrastructure at their edge locations. Each [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zone connects to an MNO's 5G or 4G network, allowing low-latency connections between applications running in the zone and mobile devices.

## Comparison & Anti-Patterns

The table below compares [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]:

| Service               | Description                                                                   |
|-----------------------|-------------------------------------------------------------------------------|
| [[Collab_Notes_detailed/Core_Compute/Wavelength|Wavelength]] Zones      | Single-digit millisecond latency for mobile applications via MNO partnerships.|
| Outposts             | Extend AWS infrastructure to your data center or co-location space.       |
| Local Zones           | Low-latency compute, storage, and database services for select metro areas.|
| Global Accelerator    | Improve application performance using AWS-owned points of presence (PoPs).|

Anti-patterns include:

- Using [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones for non-mobile use cases.
- Implementing [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones without considering throttling limits and exponential backoff strategies.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] require careful consideration when implementing [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones. JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "wavelength:Describe*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",