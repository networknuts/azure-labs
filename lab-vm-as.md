# AZ-104 Lab Instructions: Deploy Linux VM in an Availability Set (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create an Availability Set with 3 Fault Domains and 2 Update Domains  
3. Deploy a Linux virtual machine (VM) into that Availability Set  
4. Verify the Availability Set and VM configuration  

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Availability Sets, and Virtual Machines  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value                                      |
|-------------------------------|--------------------------------------------|
| **Region**                    | East US                                    |
| **Resource Group Name**       | az104-as-rg                                |
| **Availability Set Name**     | linux-as                                  |
| **Fault Domain Count**        | 3                                          |
| **Update Domain Count**       | 2                                          |
| **Linux VM Name**             | linux-as-vm                                |
| **Linux VM Admin Username**   | azureuser                                  |
| **SSH Public Key**            | Paste contents of `~/.ssh/id_rsa.pub`      |
| **Linux VM Image**            | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **Linux VM Size**             | Standard_DS1_v2                            |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-as-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Create Availability Set

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for and select **Availability set**.  
3. Click **Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-as-rg**  
   - **Name**: `linux-as`  
   - **Region**: `East US`  
   - **Platform fault domains**: `3`  
   - **Platform update domains**: `2`  
   - **Managed**: Leave as **Yes** (default)  
5. Click **Review + create**, then **Create**.  
6. Wait until you see **“Deployment succeeded”**.  
7. Navigate to **Resource groups** → **az104-as-rg** → **linux-as** to confirm:  
   - **Fault domains** = 3  
   - **Update domains** = 2  

   Take a screenshot of the Availability Set’s **Overview** page showing these values.

---

## Task 3: Deploy Linux VM into Availability Set

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine**.  
3. Under the **Basics** tab, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-as-rg`  
   - **Virtual machine name**: `linux-as-vm`  
   - **Region**: `East US`  
   - **Availability options**: Choose **Availability set**, then select **linux-as**  
   - **Image**: Ubuntu Server 22.04 LTS (UbuntuLTS)  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of your `~/.ssh/id_rsa.pub` file  
4. Under **Inbound ports**, ensure **SSH (22)** is selected.  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD for OS), then **Next: Networking**.  
6. Under **Networking**, accept all defaults (new VNet, subnet, public IP, Basic NIC NSG allowing SSH).  
7. Click **Next: Management**, leave defaults.  
8. Click **Next: Advanced**, leave defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings match the variables above (especially that **Availability set** = `linux-as`), then **Create**.  
11. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 4: Verify VM and Availability Set Configuration

1. In the Azure Portal, navigate to **Virtual machines** and select **linux-as-vm**.  
2. On the **Overview** page, verify the **Availability set** field displays **linux-as**.  
   - Take a screenshot showing the **Availability set** value under **linux-as-vm → Overview**.  
3. In the **Availability set** blade (`az104-as-rg` → **linux-as**), click **Virtual machines**.  
4. Confirm that **linux-as-vm** appears as one of the members of the availability set.  
   - Take a screenshot of the **Virtual machines** list under the **linux-as** Availability Set, showing **linux-as-vm**.  

---

## Deliverables

1. **Availability Set Overview Screenshot**  
   - Show **linux-as** Availability Set in **East US** with **Fault domains = 3** and **Update domains = 2**.  

2. **VM Deployment Screenshot**  
   - Show **linux-as-vm** in **East US** with **Availability set = linux-as** on the VM’s **Overview** page.  

3. **Availability Set VM List Screenshot**  
   - Show the **linux-as** Availability Set’s **Virtual machines** list containing **linux-as-vm**.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-as-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-as-rg`** to confirm, then click **Delete**.  

> This action permanently deletes the Availability Set, VM, managed disks, network interfaces, public IPs, and any other resources under **`az104-as-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, fault/update domain counts, usernames, and images—any deviation will affect grading.  
- Ensure that when creating the VM, you explicitly set **Availability options** to **Availability set** and choose **linux-as**.  
- If a VM appears outside the availability set, delete and recreate with the correct Availability Set selection.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation or management.  

Good luck completing the lab tasks!  
