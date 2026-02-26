
**1 [[VPN]] uses internet**
**2. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Accelerated Site-to-Site VPN]]**

• **The Problem:** Standard [[VPN]] traffic travels over the public internet, hopping between various Internet Service Providers (ISPs). This path is unpredictable and prone to congestion (latency spikes).

• **The Solution:** Accelerated [[VPN]] uses **[[AWS Global Accelerator 1]]**.

• **How it Works:**

    1. Traffic leaves your on-premises router.

    2. Instead of traveling the whole way to the AWS Region over the internet, it enters the **AWS Global Network** at the Edge Location closest to you.

    3. It rides the dedicated, congestion-free AWS fiber backbone to the target [[Transit gateway]].

• **Result:** More consistent latency and fewer packet drops compared to standard [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]].