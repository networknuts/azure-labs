
# AZ-104 Lab Instructions: Deploy Azure File Share and Connect Multiple Linux VMs (NAS)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a Resource Group  
2. Create a Storage Account with an Azure File Share  
3. Deploy two Linux virtual machines (VMs) without public IPs  
4. Install and configure SMB client tools on each VM to mount the Azure File Share as a network drive (NAS)  
5. Verify both VMs can read/write to the same Azure File Share  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface, except where instructed to SSH into the VMs to install and configure SMB/NFS tools._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Storage Accounts, and Virtual Machines  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  
- Familiarity with installing Linux packages (`cifs-utils`) and editing `/etc/fstab`  

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value                                       |
|-------------------------------|---------------------------------------------|
| **Region**                    | East US                                     |
| **Resource Group Name**       | az104-fileshare-rg                          |
| **Storage Account Name**      | az104fileshareacct (must be globally unique)|
| **Storage SKU**               | Standard_LRS                                |
| **File Share Name**           | az104-share                                 |
| **Linux VM 1 Name**           | fileshare-vm1                               |
| **Linux VM 2 Name**           | fileshare-vm2                               |
| **Admin Username**            | azureuser                                   |
| **SSH Public Key**            | Paste contents of `~/.ssh/id_rsa.pub`       |
| **Linux VM Image**            | Ubuntu Server 22.04 LTS (UbuntuLTS)         |
| **Linux VM Size**             | Standard_DS1_v2                             |
| **VNet Name**                 | az104-vnet                                  |
| **VNet Address Space**        | 10.0.0.0/16                                 |
| **Subnet Name**               | default                                     |
| **Subnet Prefix**             | 10.0.0.0/24                                 |
| **NSG Name**                  | az104-fileshare-nsg                         |
| **Mount Point Path**          | `/mnt/azfileshare`                          |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then click **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.  

---

## Task 2: Create Storage Account and File Share

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for **Storage account**, then click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Storage account name**: `az104fileshareacct` (must be unique)  
   - **Region**: `East US`  
   - **Performance**: `Standard`  
   - **Replication**: `Locally-redundant storage (LRS)`  
4. Click **Next: Advanced**, leave defaults (default is fine).  
5. Click **Next: Networking**.  
6. Under **Connectivity method**, select **Enabled from selected virtual networks and IP addresses** (to restrict access to the VNet created later).  
7. Click **Next: Data protection**, leave defaults.  
8. Click **Next: Encryption**, leave defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings match above, then click **Create**.  
11. Wait for **“Deployment succeeded”**.  
12. Navigate to **Resource groups** → **az104-fileshare-rg** → **az104fileshareacct**.  
13. In the storage account blade, click **File shares** under **Data storage**, then click **+ File share**.  
14. Under **Create file share**, configure:  
    - **Name**: `az104-share`  
    - **Quota (GiB)**: `5` (default is fine)  
15. Click **Create**.  
16. Confirm **az104-share** appears in the list.  
17. Under **az104-share**, click **Connect** (upper toolbar).  
18. In the **Connect file share** pane, select **Linux**.  
19. Copy the **Mount command** template (it contains `<storage-account-name>` and `<storage-account-key>`)—you will use this on both VMs. Take a screenshot of the command.  

---

## Task 3: Create Virtual Network and NSG

1. In the Azure Portal, click **Create a resource**, search for **Virtual network**, then click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Name**: `az104-vnet`  
   - **Region**: `East US`  
3. Click **Next: IP Addresses**.  
4. Under **IP Addresses**, configure:  
   - **IPv4 address space**: `10.0.0.0/16`  
5. Under **Subnets**, ensure one default subnet with:  
   - **Subnet name**: `default`  
   - **Subnet prefix**: `10.0.0.0/24`  
6. Click **Review + create**, verify settings, then click **Create**.  
7. Wait for **“Deployment succeeded”**.  
8. In the Azure Portal, click **Create a resource**, search for **Network security group**, then click **Create**.  
9. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Name**: `az104-fileshare-nsg`  
   - **Region**: `East US`  
10. Click **Review + create**, then click **Create**.  
11. Wait for **“Deployment succeeded”**.  
12. Navigate to **Resource groups** → **az104-fileshare-rg** → **az104-fileshare-nsg**.  
13. Click **Inbound security rules** → **+ Add**.  
14. Configure a rule to allow SSH only from your corporate/public IP (for bastion/Cloud Shell):  
    - **Source**: `IP Addresses`  
    - **Source IP addresses/CIDR ranges**: your office or home public IP (e.g., `YOUR.IP.ADD.RESS/32`)  
    - **Source port ranges**: `*`  
    - **Destination**: `Any`  
    - **Destination port ranges**: `22`  
    - **Protocol**: `TCP`  
    - **Action**: `Allow`  
    - **Priority**: `100`  
    - **Name**: `Allow-SSH-From-MyIP`  
15. Click **Add**.  
16. Leave **Outbound security rules** at defaults (allow all).

---

## Task 4: Deploy Two Linux VMs into VM Subnet (No Public IP)

You will create two Ubuntu VMs in the `az104-vnet/default` subnet, each with no public IP. Both VMs will mount the same Azure File Share.

### 4.1 Deploy VM1

1. In the Azure Portal, click **Virtual machines** → **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Virtual machine name**: `fileshare-vm1`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required**  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Admin username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, set **Public inbound ports** to **None**.  
4. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD).  
5. Click **Next: Networking**.  
6. Under **Networking**, configure:  
   - **Virtual network**: `az104-vnet`  
   - **Subnet**: `default`  
   - **Public IP**: **None**  
   - **Network security group**: **Basic** → Select **az104-fileshare-nsg**  
7. Click **Next: Management**, leave defaults.  
8. Click **Next: Advanced**, leave defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings, then click **Create**.  
11. Wait for **“Deployment succeeded”**.  
12. Navigate to **Resource groups** → **az104-fileshare-rg** → **fileshare-vm1** and confirm:  
    - **No Public IP**  
    - **Subnet = default**  
    - **NSG = az104-fileshare-nsg**  
    - Take a screenshot of the VM’s **Overview** page.

---

### 4.2 Deploy VM2

1. In the Azure Portal, click **Virtual machines** → **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fileshare-rg`  
   - **Virtual machine name**: `fileshare-vm2`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required**  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: **Standard_DS1_v2**  
   - **Authentication type**: SSH public key  
   - **Admin username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, set **Public inbound ports** to **None**.  
4. Click **Next: Disks**, leave defaults.  
5. Click **Next: Networking**.  
6. Under **Networking**, configure:  
   - **Virtual network**: `az104-vnet`  
   - **Subnet**: `default`  
   - **Public IP**: **None**  
   - **Network security group**: **Basic** → Select **az104-fileshare-nsg**  
7. Click **Next: Management**, leave defaults.  
8. Click **Next: Advanced**, leave defaults.  
9. Click **Next: Tags**, leave blank.  
10. Click **Next: Review + create**, verify all settings, then click **Create**.  
11. Wait for **“Deployment succeeded”**.  
12. Navigate to **Resource groups** → **az104-fileshare-rg** → **fileshare-vm2** and confirm:  
    - **No Public IP**  
    - **Subnet = default**  
    - **NSG = az104-fileshare-nsg**  
    - Take a screenshot of the VM’s **Overview** page.

---

## Task 5: Mount Azure File Share on Each VM

You will SSH into each VM (via Bastion or Cloud Shell) to install SMB tools and mount the Azure File Share at `/mnt/azfileshare`.

### 5.1 Retrieve Storage Account Key

1. In the Azure Portal, navigate to **Resource groups** → **az104-fileshare-rg** → **az104fileshareacct**.  
2. In the storage account blade, click **Access keys** under **Security + networking**.  
3. Click **Show keys** and copy **key1** to a safe place (you will need it for mounting).  
4. Take a screenshot of the key (obscure part of the key or redact it if necessary).

---

### 5.2 SSH into VM1 and Mount File Share

1. In the Azure Portal, navigate to **Resource groups** → **az104-fileshare-rg** → **fileshare-vm1**.  
2. On **fileshare-vm1**’s **Overview** page, click **Connect** → **Bastion** (or use Cloud Shell’s SSH).  
3. In the Bastion pane (or SSH session), authenticate with:  
   - **Username**: `azureuser`  
   - **SSH private key**: your private key (`~/.ssh/id_rsa`)  
4. Once connected, run the following commands:  
   ```bash
   # Install cifs-utils for SMB mount
   sudo apt update
   sudo apt install -y cifs-utils

   # Create mount point
   sudo mkdir -p /mnt/azfileshare

   # Mount Azure File Share via SMB
   sudo mount -t cifs //az104fileshareacct.file.core.windows.net/az104-share /mnt/azfileshare \
     -o vers=3.0,username=az104fileshareacct,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino

   # Verify mount
   df -h | grep az104-share
   ```

* Replace `<storage-account-key>` with the key you copied.
* Take a screenshot showing `df -h` with `/mnt/azfileshare` mounted.

5. (Optional) Make mount persistent:

   ```bash
   echo "//az104fileshareacct.file.core.windows.net/az104-share /mnt/azfileshare cifs vers=3.0,username=az104fileshareacct,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino" | \
     sudo tee -a /etc/fstab
   sudo mount -a
   ```

   * Take a screenshot showing the `/etc/fstab` entry.

6. Create a test file on the share:

   ```bash
   echo "Hello from VM1" | sudo tee /mnt/azfileshare/vm1.txt
   ls /mnt/azfileshare
   ```

   * Take a screenshot showing `vm1.txt` in the share.

7. Exit the SSH session.

---

### 5.3 SSH into VM2 and Mount File Share

1. In the Azure Portal, navigate to **Resource groups** → **az104-fileshare-rg** → **fileshare-vm2**.

2. On **fileshare-vm2**’s **Overview** page, click **Connect** → **Bastion** (or use Cloud Shell’s SSH).

3. In the Bastion pane (or SSH session), authenticate with:

   * **Username**: `azureuser`
   * **SSH private key**: your private key (`~/.ssh/id_rsa`)

4. Once connected, run:

   ```bash
   # Install cifs-utils
   sudo apt update
   sudo apt install -y cifs-utils

   # Create mount point
   sudo mkdir -p /mnt/azfileshare

   # Mount Azure File Share via SMB
   sudo mount -t cifs //az104fileshareacct.file.core.windows.net/az104-share /mnt/azfileshare \
     -o vers=3.0,username=az104fileshareacct,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino

   # Verify mount
   df -h | grep az104-share
   ```

   * Replace `<storage-account-key>` with the same key.
   * Take a screenshot showing `df -h` with `/mnt/azfileshare` mounted.

5. (Optional) Add to `/etc/fstab` as in VM1 to persist across reboots.

6. List contents of the share and verify `vm1.txt` exists:

   ```bash
   ls /mnt/azfileshare
   ```

   * Take a screenshot showing `vm1.txt`.

7. Create a second test file:

   ```bash
   echo "Hello from VM2" | sudo tee /mnt/azfileshare/vm2.txt
   ls /mnt/azfileshare
   ```

   * Take a screenshot showing both `vm1.txt` and `vm2.txt` in the share.

8. Exit the SSH session.

---

## Task 6: Verify Shared Access from VM1

1. SSH back into **fileshare-vm1** via Bastion:

   ```bash
   ssh azureuser@<bastion-or-cloudshell>
   ```
2. Verify both test files created by VM1 and VM2 exist:

   ```bash
   ls /mnt/azfileshare
   cat /mnt/azfileshare/vm2.txt
   ```

   * Take a screenshot showing `vm1.txt`, `vm2.txt` and the contents of `vm2.txt`.
3. Create a third test file:

   ```bash
   echo "Combined access test" | sudo tee /mnt/azfileshare/test-vm1.txt
   ```

   * Take a screenshot showing `test-vm1.txt` in the share.
4. Exit the SSH session.

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-fileshare-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **Storage Account & File Share Screenshots**

   * **az104fileshareacct Overview** showing **Resource group = az104-fileshare-rg**, **Region = East US**.
   * **az104-share** under **File shares**, showing the share exists.
   * **Connect file share (Linux)** pane showing the mount command.

3. **VNet & NSG Screenshots**

   * **az104-vnet** with **Address space = 10.0.0.0/16**, **Subnet = default (10.0.0.0/24)**.
   * **az104-fileshare-nsg** inbound rule: **Allow-SSH-From-MyIP**.

4. **VM Overview Screenshots**

   * **fileshare-vm1**: Show **No Public IP**, **Subnet = default**, **NSG = az104-fileshare-nsg**.
   * **fileshare-vm2**: Show **No Public IP**, **Subnet = default**, **NSG = az104-fileshare-nsg**.

5. **Mount Verification Screenshots**

   * On **fileshare-vm1**, show `df -h` with `/mnt/azfileshare` mounted.
   * On **fileshare-vm1**, show `ls /mnt/azfileshare` listing `vm1.txt`.
   * On **fileshare-vm2**, show `df -h` with `/mnt/azfileshare` mounted.
   * On **fileshare-vm2**, show `ls /mnt/azfileshare` listing both `vm1.txt` and `vm2.txt`.
   * On **fileshare-vm1** again, show `ls /mnt/azfileshare` listing all three test files.

6. **/etc/fstab Persistence Screenshots (Optional)**

   * Show the `/etc/fstab` entry on either VM if you added it.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-fileshare-rg**.
3. Click **Delete resource group**.
4. Type **`az104-fileshare-rg`** to confirm, then click **Delete**.

> This action permanently deletes the Storage Account, File Share, VMs, VNet, NSG, and any other resources under **`az104-fileshare-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, SKU, and prefixes—any deviation will affect grading.
* When creating the Storage Account, ensure **Networking** is set to allow access from the VNet (or you may choose “All networks” if you prefer open access).
* Use the **Connect file share (Linux)** pane to get the correct SMB mount command, including the storage account key.
* Ensure the NSG on the VM subnet allows SSH only from your IP—verify by SSH’ing from your home/office network.
* If the mount fails due to permissions, confirm `cifs-utils` is installed and that the storage key is correct.
* All steps (except SSH into VMs) must be performed through the Azure Portal GUI.

Good luck completing the lab tasks!
