# AZ-104 Lab Instructions: Create Two VNets and Configure VNet Peering (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create a Virtual Network (VNet1) with CIDR `10.1.0.0/16`  
3. Create a second Virtual Network (VNet2) with CIDR `192.168.0.0/16`  
4. Configure bidirectional VNet Peering between VNet1 and VNet2  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups and Virtual Networks  
- A user account that can log into the Azure Portal  

---

## Lab Variables (Use Exactly These Values)

| Variable                       | Value                   |
|--------------------------------|-------------------------|
| **Region**                     | East US                 |
| **Resource Group Name**        | az104-peering-rg        |
| **VNet1 Name**                 | vnet-primary            |
| **VNet1 Address Space**        | 10.1.0.0/16             |
| **VNet1 Subnet Name**          | Subnet-Primary          |
| **VNet1 Subnet Prefix**        | 10.1.1.0/24             |
| **VNet2 Name**                 | vnet-secondary          |
| **VNet2 Address Space**        | 192.168.0.0/16          |
| **VNet2 Subnet Name**          | Subnet-Secondary        |
| **VNet2 Subnet Prefix**        | 192.168.1.0/24          |
| **Peering Name (from VNet1)**  | Peer-PrimaryToSecondary |
| **Peering Name (from VNet2)**  | Peer-SecondaryToPrimary |

_Do not change any of these values. If you modify names, regions, or address spaces, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-peering-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Create VNet1 (vnet-primary)

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for **Virtual network**, then click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-peering-rg**  
   - **Name**: `vnet-primary`  
   - **Region**: `East US`  
4. Click **Next: IP Addresses**.  
5. Under **IP Addresses**, set:  
   - **IPv4 address space**: `10.1.0.0/16`  
6. Under **Subnets**, delete any existing default. Then click **+ Add subnet**:  
   - **Subnet name**: `Subnet-Primary`  
   - **Subnet address range**: `10.1.1.0/24`  
   - Click **Add**.  
7. Click **Review + create**, verify **Address space** = `10.1.0.0/16` and **Subnet** = `10.1.1.0/24`, then click **Create**.  
8. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Create VNet2 (vnet-secondary)

1. In the Azure Portal, click **Create a resource**, search for **Virtual network**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-peering-rg**  
   - **Name**: `vnet-secondary`  
   - **Region**: `East US`  
3. Click **Next: IP Addresses**.  
4. Under **IP Addresses**, set:  
   - **IPv4 address space**: `192.168.0.0/16`  
5. Under **Subnets**, delete any existing default. Then click **+ Add subnet**:  
   - **Subnet name**: `Subnet-Secondary`  
   - **Subnet address range**: `192.168.1.0/24`  
   - Click **Add**.  
6. Click **Review + create**, verify **Address space** = `192.168.0.0/16` and **Subnet** = `192.168.1.0/24`, then click **Create**.  
7. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 4: Configure VNet Peering from VNet1 to VNet2

1. In the Azure Portal, navigate to **Resource groups** → **az104-peering-rg** → **vnet-primary**.  
2. In the left-hand menu of **vnet-primary**, click **Peerings**.  
3. Click **+ Add**.  
4. Under **Basics**, configure:  
   - **Peering link name (this virtual network)**: `Peer-PrimaryToSecondary`  
   - **Peer details**:  
     - **Subscription**: Same subscription  
     - **Virtual network**: Select **vnet-secondary**  
   - **Name (remote virtual network)**: The portal will auto-fill, ensure it is `Peer-SecondaryToPrimary`  
5. Under **Configuration**, leave all defaults (both **Allow virtual network access** and **Allow forwarded traffic** should be **Enabled**).  
6. Click **Add**.  
7. Wait until you see **“Peering created”**.

---

## Task 5: Configure VNet Peering from VNet2 to VNet1

1. In the Azure Portal, navigate to **Resource groups** → **az104-peering-rg** → **vnet-secondary**.  
2. In the left-hand menu of **vnet-secondary**, click **Peerings**.  
3. You will see the incoming peering request or click **+ Add** if needed.  
4. Under **Basics**, configure:  
   - **Peering link name (this virtual network)**: `Peer-SecondaryToPrimary`  
   - **Peer details**:  
     - **Subscription**: Same subscription  
     - **Virtual network**: Select **vnet-primary**  
   - **Name (remote virtual network)**: Ensure it is `Peer-PrimaryToSecondary`  
5. Under **Configuration**, leave all defaults (verify **Allow virtual network access** is **Enabled**).  
6. Click **Add**.  
7. Wait until you see **“Peering created”**.  

> **Note:** In some cases, Azure auto-completes the reverse peering. If the reverse peering appears automatically with correct names and settings, you do not need to create a second peering manually.  

---

## Task 6: Verify VNet Peering Status

1. In the Azure Portal, navigate to **Resource groups** → **az104-peering-rg** → **vnet-primary** → **Peerings**.  
2. Confirm that you see a peering named **Peer-PrimaryToSecondary** with **Peering status** = **Connected**.  
3. In the Azure Portal, navigate to **Resource groups** → **az104-peering-rg** → **vnet-secondary** → **Peerings**.  
4. Confirm that you see a peering named **Peer-SecondaryToPrimary** with **Peering status** = **Connected**.  
5. Take screenshots of both peering blades showing **Connected** status.

---

## Deliverables

1. **Resource Group Screenshot**  
   - Show **`az104-peering-rg`** in **East US** with provisioning state **“Succeeded.”**  

2. **VNet1 Overview Screenshot**  
   - Show **`vnet-primary`** with **Address space = 10.1.0.0/16** and **Subnet-Primary (10.1.1.0/24)**.  

3. **VNet2 Overview Screenshot**  
   - Show **`vnet-secondary`** with **Address space = 192.168.0.0/16** and **Subnet-Secondary (192.168.1.0/24)**.  

4. **VNet1 Peering Screenshot**  
   - Show **Peer-PrimaryToSecondary** under **vnet-primary → Peerings** with **Peering status = Connected**.  

5. **VNet2 Peering Screenshot**  
   - Show **Peer-SecondaryToPrimary** under **vnet-secondary → Peerings** with **Peering status = Connected**.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-peering-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-peering-rg`** to confirm, then click **Delete**.  

> This action permanently deletes both VNets and all associated resources under **`az104-peering-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, address spaces, subnet prefixes, and peering names—any deviation will affect grading.  
- When creating each VNet, ensure you remove the default subnet and replace it with the specified subnet name and address prefix.  
- When configuring peering, double-check that **Allow virtual network access** is **Enabled** on both sides.  
- If you see the reverse peering auto-populated, verify its name and settings rather than creating a duplicate.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.  

Good luck completing the lab tasks!  
