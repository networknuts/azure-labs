
# AZ-104 Lab Instructions: Create VNet with Two Subnets and Linux VMs (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create a Virtual Network with two subnets  
3. Deploy one Linux VM in each subnet without public IPs  
4. Verify connectivity by pinging between the two Linux VMs  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Virtual Networks, and Virtual Machines  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                  | Value                         |
|---------------------------|-------------------------------|
| **Region**                | East US                       |
| **Resource Group Name**   | az104-net-rg                  |
| **Virtual Network Name**  | az104-vnet                    |
| **VNet Address Space**    | 10.0.0.0/16                   |
| **Subnet 1 Name**         | Subnet1                       |
| **Subnet 1 Prefix**       | 10.0.1.0/24                   |
| **Subnet 2 Name**         | Subnet2                       |
| **Subnet 2 Prefix**       | 10.0.2.0/24                   |
| **NSG Name**              | az104-nsg                     |
| **Linux VM 1 Name**       | vm-subnet1                    |
| **Linux VM 2 Name**       | vm-subnet2                    |
| **Admin Username**        | azureuser                     |
| **SSH Public Key**        | Paste contents of `~/.ssh/id_rsa.pub` |
| **Linux VM Image**        | Ubuntu Server 22.04 LTS (UbuntuLTS) |
| **Linux VM Size**         | Standard_DS1_v2               |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-net-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then click **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.  

---

## Task 2: Create Virtual Network and Subnets

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for **Virtual network**, then click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-net-rg**  
   - **Name**: `az104-vnet`  
   - **Region**: `East US`  
4. Click **Next: IP Addresses**.  
5. Under **IP Addresses**, set:  
   - **IPv4 address space**: `10.0.0.0/16`  
6. Under **Subnets**, delete any default subnet if present. Then click **+ Add subnet** twice:  

   1. **First Subnet (Subnet1)**:  
      - **Subnet name**: `Subnet1`  
      - **Subnet address range**: `10.0.1.0/24`  
      - Click **Add**.  

   2. **Second Subnet (Subnet2)**:  
      - Click **+ Add subnet** again.  
      - **Subnet name**: `Subnet2`  
      - **Subnet address range**: `10.0.2.0/24`  
      - Click **Add**.  

7. Click **Review + create**, verify **Address space** = `10.0.0.0/16` and both subnets appear, then click **Create**.  
8. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Create Network Security Group (NSG)

1. In the Azure Portal, click **Create a resource**, search for **Network security group**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-net-rg**  
   - **Name**: `az104-nsg`  
   - **Region**: `East US`  
3. Click **Review + create**, then **Create**.  
4. Wait for **“Deployment succeeded”** before proceeding.  
5. Navigate to **Resource groups** → **az104-net-rg** → **az104-nsg**.  
6. In the NSG blade, click **Inbound security rules**, then click **+ Add**.  
7. Add a rule to allow ICMP (ping) within the VNet:  
   - **Source**: `VirtualNetwork`  
   - **Source port ranges**: `*`  
   - **Destination**: `VirtualNetwork`  
   - **Destination port ranges**: `*`  
   - **Protocol**: `Any`  
   - **Action**: `Allow`  
   - **Priority**: `100`  
   - **Name**: `Allow-VNet-ICMP`  
   - Click **Add**.  
8. Repeat similar steps under **Outbound security rules** (if needed) to allow ICMP to `VirtualNetwork`, but outbound VirtualNetwork-to-VirtualNetwork is allowed by default.  

---

## Task 4: Associate NSG to Both Subnets

1. In the Azure Portal, click **Resource groups**, select **az104-net-rg**, then click **az104-vnet** under Resources.  
2. In the **az104-vnet** blade, click **Subnets**.  
3. Select **Subnet1**, then under **Network security group**, click **Associate**, select **az104-nsg**, and click **Save**.  
4. Select **Subnet2**, then click **Associate**, select **az104-nsg**, and click **Save**.  
5. Confirm both subnets show **az104-nsg** under the **Network security group** column.

---

## Task 5: Deploy Linux VM in Subnet1

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-net-rg`  
   - **Virtual machine name**: `vm-subnet1`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required** (default)  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste contents of `~/.ssh/id_rsa.pub`  
4. Under **Inbound ports**, **uncheck** all (ensure **Public inbound ports** = **None**).  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD).  
6. Click **Next: Networking**.  
7. Under **Networking**, configure:  
   - **Virtual network**: Select **az104-vnet**  
   - **Subnet**: Select **Subnet1**  
   - **Public IP**: **None**  
   - **Network security group**: **Basic** (the NSG is already associated with the subnet)  
8. Click **Next: Management**, leave defaults.  
9. Click **Next: Advanced**, leave defaults.  
10. Click **Next: Tags**, leave blank.  
11. Click **Next: Review + create**, verify all settings match the variables above, then **Create**.  
12. Wait for **“Deployment succeeded”** before proceeding.  
13. Navigate to **Resource groups** → **az104-net-rg** → **vm-subnet1** and confirm:  
    - **No Public IP address** is listed  
    - **Subnet** = **Subnet1**  
    - Take a screenshot of the VM’s **Overview** page.

---

## Task 6: Deploy Linux VM in Subnet2

1. In the Azure Portal, click **Virtual machines** and then **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-net-rg`  
   - **Virtual machine name**: `vm-subnet2`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required** (default)  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: **Standard_DS1_v2**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste contents of `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, **uncheck** all (ensure **Public inbound ports** = **None**).  
4. Click **Next: Disks**, leave defaults.  
5. Click **Next: Networking**.  
6. Under **Networking**, configure:  
   - **Virtual network**: Select **az104-vnet**  
   - **Subnet**: Select **Subnet2**  
   - **Public IP**: **None**  
   - **Network security group**: **Basic** (subnet-level NSG is already applied)  
7. Click **Next: Management**, leave defaults.  
8. Click **Next: Advanced**, leave defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings match the variables above, then **Create**.  
11. Wait for **“Deployment succeeded”** before proceeding.  
12. Navigate to **Resource groups** → **az104-net-rg** → **vm-subnet2** and confirm:  
    - **No Public IP address** is listed  
    - **Subnet** = **Subnet2**  
    - Take a screenshot of the VM’s **Overview** page.

---

## Task 7: Verify Connectivity by Pinging Between VMs

1. In the Azure Portal, navigate to **Resource groups** → **az104-net-rg** → **vm-subnet1**.  
2. On **vm-subnet1**’s **Overview** page, click **Connect** → **Bastion** (or use **Run Command → RunShellScript** if Bastion is not configured).  
3. If using **Bastion**, select the bastion host (if configured) and authenticate with:  
   - **Username**: `azureuser`  
   - **SSH private key**: your private key (`~/.ssh/id_rsa`)  
4. Once connected to **vm-subnet1**, run:  
   ```bash
   ip addr show eth0 | grep "inet "
   ```

* Note the **Private IP address** (e.g., `10.0.1.4`).
* Take a screenshot of the output.

5. In the same SSH session (on **vm-subnet1**), run:

   ```bash
   ping -c 4 10.0.2.4
   ```

   * Replace `10.0.2.4` with the actual private IP of **vm-subnet2** (obtainable from the VM’s **Overview** page or via Run Command).
   * Confirm you receive ping replies.
   * Take a screenshot of the successful ping output.

6. (Optional) Repeat steps 1–5 for **vm-subnet2** pinging **vm-subnet1** to verify bidirectional connectivity.

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-net-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **VNet and Subnets Screenshot**

   * Show **`az104-vnet`** with two subnets: **Subnet1 (10.0.1.0/24)** and **Subnet2 (10.0.2.0/24)**.

3. **NSG and Rule Screenshot**

   * Show **`az104-nsg`** Inbound security rules including **Allow-VNet-ICMP**.

4. **VM Overview Screenshots**

   * **vm-subnet1**: Show **No Public IP** and **Subnet = Subnet1**.
   * **vm-subnet2**: Show **No Public IP** and **Subnet = Subnet2**.

5. **Ping Verification Screenshot**

   * On **vm-subnet1**, show `ping -c 4 <vm-subnet2-private-ip>` with successful replies.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-net-rg**.
3. Click **Delete resource group**.
4. Type **`az104-net-rg`** to confirm, then click **Delete**.

> This action permanently deletes the Virtual Network, NSG, both VMs, managed disks, network interfaces, and any other resources under **`az104-net-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, address spaces, prefixes, usernames, and image—any deviation will affect grading.
* Ensure the NSG allows ICMP between subnets by creating the **Allow-VNet-ICMP** rule.
* When creating each VM, explicitly set **Public inbound ports** to **None**; both VMs must be private.
* To find a VM’s private IP, check the VM’s **Overview** page or use `ip addr` inside the VM.
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.

Good luck completing the lab tasks!
