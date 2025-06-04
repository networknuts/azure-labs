# AZ-104 Lab Instructions: Deploy Azure Bastion and a Linux VM without Public IP (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create a Virtual Network with two subnets (one for Bastion, one for the Linux VM)  
3. Deploy an Azure Bastion host into the designated Bastion subnet  
4. Deploy a Linux virtual machine (VM) into the private subnet without assigning a public IP  
5. Connect to the Linux VM through Azure Bastion and verify SSH access  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Virtual Networks, Subnets, Bastion Hosts, and VMs  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                    | Value                                      |
|-----------------------------|--------------------------------------------|
| **Region**                  | East US                                    |
| **Resource Group Name**     | az104-bastion-rg                           |
| **Virtual Network Name**    | az104-bastion-vnet                         |
| **VNet Address Space**      | 10.0.0.0/16                                |
| **Bastion Subnet Name**     | AzureBastionSubnet                         |
| **Bastion Subnet Prefix**   | 10.0.0.0/27                                |
| **VM Subnet Name**          | VMSubnet                                   |
| **VM Subnet Prefix**        | 10.0.1.0/24                                |
| **Bastion Host Name**       | az104-bastion                              |
| **Linux VM Name**           | bastion-linux-vm                           |
| **Linux VM Admin Username** | azureuser                                  |
| **SSH Public Key**          | Paste contents of `~/.ssh/id_rsa.pub`      |
| **Linux VM Image**          | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **Linux VM Size**           | Standard_DS1_v2                            |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-bastion-rg`  
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
   - **Resource group**: Select **az104-bastion-rg**  
   - **Name**: `az104-bastion-vnet`  
   - **Region**: `East US`  
4. Click **Next: IP Addresses**.  
5. Under **IP Addresses**, configure:  
   - **IPv4 address space**: `10.0.0.0/16`  
6. Under **Subnets**, delete any default subnet entry. Then click **+ Add subnet** twice:  

   1. **First Subnet (Bastion)**:  
      - **Subnet name**: `AzureBastionSubnet`  
      - **Subnet address range (prefix)**: `10.0.0.0/27`  
      - Click **Add**.  

   2. **Second Subnet (VM)**:  
      - Click **+ Add subnet** again.  
      - **Subnet name**: `VMSubnet`  
      - **Subnet address range (prefix)**: `10.0.1.0/24`  
      - Click **Add**.  

7. Click **Review + create**, verify **Address space** = `10.0.0.0/16` and the two subnets appear, then click **Create**.  
8. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Deploy Azure Bastion Host

1. In the Azure Portal, click **Create a resource**, search for **Bastion**, and select **Bastion (preview)** or **Bastion**.  
2. Click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-bastion-rg**  
   - **Name**: `az104-bastion`  
   - **Region**: `East US`  
4. Under **Virtual network**, click the dropdown and select **az104-bastion-vnet**.  
5. Under **Subnet**, ensure it shows **AzureBastionSubnet** (this is mandatory; Bastion will auto-select that subnet).  
6. Under **Public IP address**, click **Create new**:  
   - **Name**: `az104-bastion-ip`  
   - Leave **SKU** as **Standard** (default)  
   - Click **OK**.  
7. Click **Review + create**, verify all settings match the variables above (especially **Virtual network** and **Subnet**), then click **Create**.  
8. Wait until you see **“Deployment succeeded”**.  
9. Navigate to **Resource groups** → **az104-bastion-rg** → **az104-bastion** to confirm:  
   - **Bastion host** is deployed in `East US`  
   - **Subnet** = `AzureBastionSubnet`  
   - **Public IP** = `az104-bastion-ip`  
   - Take a screenshot of the Bastion host’s **Overview** page.

---

## Task 4: Deploy Linux VM with No Public IP

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-bastion-rg`  
   - **Virtual machine name**: `bastion-linux-vm`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required** (default)  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste contents of `~/.ssh/id_rsa.pub`  
4. Under **Inbound ports**, **uncheck** all boxes (ensure **Public inbound ports** is set to **None**).  
   - This ensures no Public IP is assigned and no direct inbound rules exist.  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD).  
6. Click **Next: Networking**.  
7. Under **Networking**, configure:  
   - **Virtual network**: Select **az104-bastion-vnet**  
   - **Subnet**: Select **VMSubnet**  
   - **Public IP**: **None** (ensure no Public IP is assigned)  
   - **NIC network security group**: Select **Basic** and verify **SSH (22)** is allowed from **AzureBastion** service tag.  
     > _Note: By default, the NSG will allow SSH from AzureBastion._  
8. Click **Next: Management**, leave defaults.  
9. Click **Next: Advanced**, leave defaults.  
10. Click **Next: Tags**, leave blank.  
11. Click **Next: Review + create**, verify all settings match the variables above (especially **Subnet** = `VMSubnet` and **Public IP** = **None**), then click **Create**.  
12. Wait until you see **“Deployment succeeded”**.  
13. Navigate to **Resource groups** → **az104-bastion-rg** → **bastion-linux-vm** to confirm:  
   - **No Public IP address** is listed  
   - **Subnet** = `VMSubnet`  
   - Take a screenshot of the VM’s **Overview** page.  

---

## Task 5: Connect to the Linux VM via Azure Bastion

1. In the Azure Portal, navigate to **Resource groups** → **az104-bastion-rg** → **bastion-linux-vm**.  
2. On the **Overview** page of **bastion-linux-vm**, click **Connect** → **Bastion**.  
3. In the **Bastion** pane, select the following and click **Connect**:  
   - **Bastion host**: `az104-bastion` (auto-selected)  
   - **Username**: `azureuser`  
   - **SSH private key**: If prompted, browse and select your private key file (e.g., `~/.ssh/id_rsa`)  
4. A new browser tab launches an SSH session into **bastion-linux-vm**.  
5. Once connected, verify you have a shell prompt.  
   - Run a simple command such as `ls /home/azureuser` to confirm connectivity.  
   - Take a screenshot showing the SSH session prompt inside the browser.  

---

## Deliverables

1. **Resource Group Screenshot**  
   - Show **`az104-bastion-rg`** in **East US** with provisioning state **“Succeeded.”**  

2. **Virtual Network Screenshot**  
   - Show **`az104-bastion-vnet`** with both subnets:  
     - `AzureBastionSubnet` (10.0.0.0/27)  
     - `VMSubnet` (10.0.1.0/24)  

3. **Bastion Host Overview Screenshot**  
   - Show **`az104-bastion`** in **East US**, **Subnet** = `AzureBastionSubnet`, **Public IP** = `az104-bastion-ip`.  

4. **Linux VM Overview Screenshot**  
   - Show **`bastion-linux-vm`** in **East US**, **Subnet** = `VMSubnet`, **Public IP** = **None**.  

5. **Bastion SSH Session Screenshot**  
   - Show an active SSH session to **`bastion-linux-vm`** via Bastion with a shell prompt.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-bastion-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-bastion-rg`** to confirm, then click **Delete**.  

> This action permanently deletes the Virtual Network (with subnets), Bastion host, Linux VM, managed disks, network interfaces, public IP, and any other resources under **`az104-bastion-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, prefixes, usernames, and image—any deviation will affect grading.  
- When creating the Virtual Network, ensure the Bastion subnet is named exactly **AzureBastionSubnet** with a `/27` prefix.  
- When creating the Linux VM, explicitly set **Public inbound ports** to **None** so no Public IP is assigned.  
- Verify that the NSG on **VMSubnet** allows SSH (port 22) from **AzureBastion** service tag (this is default).  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.  

Good luck completing the lab tasks!  
