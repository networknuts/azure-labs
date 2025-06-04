
# AZ-104 Lab Instructions: Create Linux VM with Additional Disk 

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Deploy a Linux virtual machine (VM) with default settings and attach an additional 10 GB managed data disk  
3. Initialize, format, and mount the new data disk inside the Linux VM  


---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Virtual Machines, and managed disks.  
- A user account that can log into the Azure Portal.  
- A local SSH key pair (if you do not already have one, refer to [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)).  

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value                                      |
|-------------------------------|--------------------------------------------|
| **Region**                    | East US                                    |
| **Resource Group Name**       | az104-lab-rg                               |
| **Linux VM Name**             | linux-disk-vm                              |
| **Linux VM Admin Username**   | azureuser                                  |
| **SSH Public Key**            | Paste contents of `~/.ssh/id_rsa.pub`      |
| **Linux VM Image**            | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **Linux VM Size**             | Standard_DS1_v2                            |
| **Additional Disk Size**      | 10 GB (10 GiB)                             |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-lab-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Deploy Linux VM with Additional 10 GB Disk

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine**.  
3. Under the **Basics** tab, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-lab-rg`  
   - **Virtual machine name**: `linux-disk-vm`  
   - **Region**: `East US`  
   - **Image**: Select **Ubuntu Server 22.04 LTS** (UbuntuLTS)  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of your `~/.ssh/id_rsa.pub` file  
4. Under **Inbound ports**, ensure **SSH (22)** is checked.  
5. Click **Next: Disks**.  

   **On the Disks tab:**  
   1. The OS disk is created automatically (Managed disk, Premium SSD).  
   2. Under **Data disks**, click **Create and attach a new disk**.  
   3. In the **Create managed disk** pane, set:  
      - **Name**: `linux-disk-vm-data1`  
      - **Region**: `East US`  
      - **Account type**: Leave as **Premium SSD** (default)  
      - **Size (GiB)**: `10` (this creates a 10 GB disk)  
      - **Source type**: **None (empty disk)**  
   4. Click **Review + create**, then **Create** to provision the new disk.  
   5. Back on the **Disks** tab of the VM creation wizard, verify that the new disk appears under **Data disks** with size 10 GiB.  

6. Click **Next: Networking**.  
7. Under **Networking**, configure:  
   - **Virtual network**: Accept the automatically created `linux-disk-vm-vnet`  
   - **Subnet**: Accept default (`10.0.0.0/24`)  
   - **Public IP**: Accept the new `linux-disk-vm-ip`  
   - **NIC network security group**: Select **Basic** and verify **SSH (22)** is allowed  

8. Click **Next: Management**, leave all defaults.  
9. Click **Next: Advanced**, leave all defaults.  
10. Click **Next: Tags**, leave blank.  
11. Click **Next: Review + create**.  
12. Verify each value exactly matches the variables above (name, region, image, size, SSH settings, data disk).  
13. Click **Create**.  
14. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Initialize, Format, and Mount the New Data Disk

1. In the Azure Portal, navigate to **Virtual machines**, then select **linux-disk-vm**.  
2. On the **Overview** page, copy the **Public IP address** (e.g., `20.x.x.x`).  
3. Open your local terminal (or Azure Cloud Shell) and connect via SSH:  
   ```bash
   ssh azureuser@<linux-disk-vm-public-ip>
   ```

4. Once logged in **on the VM**, perform these steps (capture evidence with screenshots):

   1. List block devices to identify the new disk (typically `/dev/sdc` or similar):

      ```bash
      lsblk
      ```
   2. Create a filesystem on the new disk (replace `/dev/sdc` with the correct device name):

      ```bash
      sudo mkfs.ext4 /dev/sdc
      ```
   3. Create a mount point directory:

      ```bash
      sudo mkdir /mnt/data1
      ```
   4. Mount the new disk:

      ```bash
      sudo mount /dev/sdc /mnt/data1
      ```
   5. Verify the mount:

      ```bash
      df -h
      ```

      Take a screenshot showing `/dev/sdc` mounted at `/mnt/data1`.
   6. (Optional) Update `/etc/fstab` to mount the disk on reboot:

      * Open `/etc/fstab` in an editor:

        ```bash
        sudo nano /etc/fstab
        ```
      * Add a line at the end:

        ```
        /dev/sdc   /mnt/data1   ext4   defaults,nofail   0   2
        ```
      * Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
      * Test by unmounting and remounting:

        ```bash
        sudo umount /mnt/data1
        sudo mount -a
        ```
      * Verify with `df -h` again (take a screenshot).

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-lab-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **Linux VM + Disk Verification**

   * Screenshot of the VM’s **Overview** page showing the attached data disk (`linux-disk-vm-data1`, 10 GiB).
   * SSH session screenshot showing the output of `lsblk` (identifying the new disk).
   * SSH session screenshot of the `mkfs.ext4` command on `/dev/sdc`.
   * SSH session screenshot showing `df -h` with `/dev/sdc` mounted at `/mnt/data1`.
   * (Optional) SSH session screenshot showing the `/etc/fstab` entry and a successful `mount -a` after reboot.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-lab-rg**.
3. Click **Delete resource group**.
4. Type **`az104-lab-rg`** to confirm, then click **Delete**.

> This action permanently deletes the VM, managed disks, network interfaces, public IPs, and any other resources under **`az104-lab-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, VM sizes, usernames, and disk size—any deviation will affect grading.
* When attaching the data disk, ensure you choose **“Create and attach a new disk”** and specify **10 GiB**.
* If you do not see the new disk inside the VM (e.g., `/dev/sdc`), verify in the Azure Portal under the VM’s **Disks** tab that the data disk is attached.
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation.

Good luck completing the lab tasks!
