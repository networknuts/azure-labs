# AZ-104 Lab Instructions: Deploy Linux VM Scale Set and Verify Auto-Scaling (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Deploy a Linux Virtual Machine Scale Set (VMSS) with two instances  
3. Verify the VMSS is running two instances  
4. Delete one VM instance from the scale set  
5. Verify the VMSS automatically replaces the deleted instance to maintain capacity of two  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, VM Scale Sets, and managed disks  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                    | Value                                      |
|-----------------------------|--------------------------------------------|
| **Region**                  | East US                                    |
| **Resource Group Name**     | az104-vmss-rg                              |
| **VMSS Name**               | linux-vmss                                 |
| **Admin Username**          | azureuser                                  |
| **SSH Public Key**          | Paste contents of `~/.ssh/id_rsa.pub`      |
| **VM Image**                | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **VM Size**                 | Standard_DS1_v2                            |
| **Initial Instance Count**  | 2                                          |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-vmss-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.  

---

## Task 2: Deploy Linux VM Scale Set (VMSS)

1. In the Azure Portal, click **Virtual machine scale sets** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine scale set**.  
3. Under the **Basics** tab, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-vmss-rg**  
   - **Virtual machine scale set name**: `linux-vmss`  
   - **Region**: `East US`  
   - **Availability zones**: Leave as **None** (the default)  
   - **Image**: Click **Change** and select **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then click **Select**  
   - **Authentication type**: SSH public key  
   - **Admin username**: `azureuser`  
   - **SSH public key source**: **Use existing key stored on your local machine**  
   - **SSH public key**: Paste contents of `~/.ssh/id_rsa.pub`  
   - **Instance count**: `2`  
4. Under **Inbound ports**, ensure **SSH (22)** is selected.  
5. Click **Next: Disks**, leave all defaults (Managed disk, Premium SSD for OS).  
6. Click **Next: Networking**, leave all defaults (new VNet, subnet, public IP per instance, Basic NIC NSG allowing SSH).  
7. Click **Next: Management**, leave all defaults.  
8. Click **Next: Advanced**, leave all defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings match the variables above, then click **Create**.  
11. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Verify Initial VMSS Instance Count

1. In the Azure Portal, navigate to **Resource groups** and select **az104-vmss-rg**.  
2. Under **Resources**, click **linux-vmss** to open the VMSS blade.  
3. In the **Overview** tab, verify the **Instance count** shows **2/2**.  
   - Take a screenshot of the Overview blade showing **“2/2 instances”**.  
4. Click **Instances** in the left-hand menu of the VMSS blade.  
5. You should see two instances listed (e.g., `linux-vmss_0` and `linux-vmss_1`).  
   - Take a screenshot showing both instance names and their statuses (Running).

---

## Task 4: Delete One VM Instance

1. On the **Instances** page of **linux-vmss**, select one of the instances (for example, `linux-vmss_0`).  
2. Click **Delete** at the top of the Instances pane.  
3. In the confirmation dialog, type **Yes** to confirm deletion, then click **Delete**.  
4. Wait a few moments—Azure will delete that instance.  

---

## Task 5: Verify Auto-Healing/Auto-Scaling

1. After deleting one instance, remain on the **Instances** page.  
2. Refresh the list until you see a new instance reinstated (for example, `linux-vmss_2` or the original instance recreated).  
3. Confirm that the **Instance count** returns to **2/2** under the Overview tab.  
   - Take a screenshot of the Overview blade showing **“2/2 instances”** again.  
4. On the **Instances** page, verify there are two instances running (e.g., `linux-vmss_1` and a new instance).  
   - Take a screenshot showing the updated instance list.

---

## Deliverables

1. **Resource Group Screenshot**  
   - Show **`az104-vmss-rg`** in **East US** with provisioning state **“Succeeded.”**  

2. **VMSS Initial Verification**  
   - Screenshot of the **linux-vmss Overview** blade showing **2/2 instances**.  
   - Screenshot of the **Instances** page listing both initial instances.  

3. **Instance Deletion and Auto-Healing**  
   - Screenshot of the **Instances** page immediately after deleting one instance (showing only one instance).  
   - Screenshot of the **linux-vmss Overview** blade showing **2/2 instances** after auto-healing.  
   - Screenshot of the **Instances** page showing the new instance list with two running instances.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-vmss-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-vmss-rg`** to confirm, then click **Delete**.  

> This action permanently deletes the VM Scale Set, its instances, managed disks, network interfaces, public IPs, and any other resources under **`az104-vmss-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, VM sizes, usernames, and instance count—any deviation will affect grading.  
- When deleting a VM instance, ensure you delete only from the **Instances** page of the VMSS blade (not from the individual VM resource).  
- Allow a few minutes for Azure to auto-heal the scale set and bring the instance count back to the desired capacity.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation or management.  

Good luck completing the lab tasks!  
