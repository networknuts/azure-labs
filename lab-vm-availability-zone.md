# AZ-104 Lab Instructions: Deploy Linux VMs Across Availability Zones 

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Deploy three Linux virtual machines (VMs), each in a different Availability Zone within the Central India region  

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups and VMs  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                    | Value                                      |
|-----------------------------|--------------------------------------------|
| **Region**                  | Central India                              |
| **Resource Group Name**     | az104-az-lab-rg                            |
| **Linux VM Admin Username** | azureuser                                  |
| **SSH Public Key**          | Paste contents of `~/.ssh/id_rsa.pub`      |
| **Linux VM Image**          | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **Linux VM Size**           | Standard_DS1_v2                            |
| **VM Name (Zone 1)**        | linux-vm-az1                               |
| **VM Name (Zone 2)**        | linux-vm-az2                               |
| **VM Name (Zone 3)**        | linux-vm-az3                               |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-az-lab-rg`  
   - **Region**: `Central India`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Deploy Linux VM in Availability Zone 1

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and select **Azure virtual machine**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-az-lab-rg`  
   - **Virtual machine name**: `linux-vm-az1`  
   - **Region**: `Central India`  
   - **Availability zone**: Select **1**  
   - **Image**: Ubuntu Server 22.04 LTS (UbuntuLTS)  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of `~/.ssh/id_rsa.pub`  
4. Under **Inbound ports**, ensure **SSH (22)** is selected.  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD), then **Next: Networking**.  
6. Under **Networking**, accept all defaults (new VNet, subnet, public IP, Basic NIC NSG allowing SSH).  
7. Click **Next: Management**, leave all defaults.  
8. Click **Next: Advanced**, leave all defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings match the variables above, then **Create**.  
11. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Deploy Linux VM in Availability Zone 2

1. In the Azure Portal, click **Virtual machines**, then **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-az-lab-rg`  
   - **Virtual machine name**: `linux-vm-az2`  
   - **Region**: `Central India`  
   - **Availability zone**: Select **2**  
   - **Image**: Ubuntu Server 22.04 LTS (UbuntuLTS)  
   - **Size**: Standard_DS1_v2 (ensure you select it under **Change size**)  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, ensure **SSH (22)** is selected.  
4. Click **Next: Disks**, leave defaults, then **Next: Networking**.  
5. Under **Networking**, accept defaults (new VNet, subnet, public IP, Basic NIC NSG allowing SSH).  
6. Click **Next: Management**, leave defaults.  
7. Click **Next: Advanced**, leave defaults.  
8. Click **Next: Tags**, leave blank.  
9. Click **Next: Review + create**, verify all settings match the variables above, then **Create**.  
10. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 4: Deploy Linux VM in Availability Zone 3

1. In the Azure Portal, click **Virtual machines**, then **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-az-lab-rg`  
   - **Virtual machine name**: `linux-vm-az3`  
   - **Region**: `Central India`  
   - **Availability zone**: Select **3**  
   - **Image**: Ubuntu Server 22.04 LTS (UbuntuLTS)  
   - **Size**: Standard_DS1_v2 (select under **Change size**)  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, ensure **SSH (22)** is selected.  
4. Click **Next: Disks**, leave defaults, then **Next: Networking**.  
5. Under **Networking**, accept defaults (new VNet, subnet, public IP, Basic NIC NSG allowing SSH).  
6. Click **Next: Management**, leave defaults.  
7. Click **Next: Advanced**, leave defaults.  
8. Click **Next: Tags**, leave blank.  
9. Click **Next: Review + create**, verify all settings match the variables above, then **Create**.  
10. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 5: Verify All Three VMs

1. In the Azure Portal, navigate to **Virtual machines**.  
2. For each VM (`linux-vm-az1`, `linux-vm-az2`, `linux-vm-az3`):  
   - Open the **Overview** page.  
   - Verify that **Region** is **Central India**.  
   - Verify that **Availability zone** is correctly set to 1, 2, or 3, respectively.  
   - Copy the **Public IP address**.  
   - Open your local terminal (or Azure Cloud Shell) and connect via SSH:  
     ```bash
     ssh azureuser@<public-ip>
     ```  
   - Confirm you can successfully authenticate and reach the VM’s shell.  
   - Take a screenshot of the VM’s **Overview** page showing the correct Availability Zone and a screenshot of the SSH session prompt on each VM.  

---

## Deliverables

1. **Resource Group Screenshot**  
   - Show `az104-az-lab-rg` in **Central India** with provisioning state **“Succeeded.”**  

2. **Linux VM Verification (for each of the three VMs)**  
   - Screenshot of each VM’s **Overview** page showing:  
     - Region: Central India  
     - Availability Zone: 1 (for `linux-vm-az1`), 2 (for `linux-vm-az2`), or 3 (for `linux-vm-az3`)  
   - SSH session screenshot proving you can connect to each VM (show the shell prompt with the correct VM hostname).  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-az-lab-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-az-lab-rg`** to confirm, then click **Delete**.  

> This action permanently deletes all three VMs, associated disks, NICs, public IPs, and any other resources under **`az104-az-lab-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, VM sizes, usernames, and Availability Zones—any deviation will affect grading.  
- Ensure you explicitly select Availability Zone 1, 2, or 3 under the **Basics** tab for each VM.  
- If you do not see the VM in the correct Availability Zone, delete and recreate with the correct zone selection.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation.  

Good luck completing the lab tasks!  
