# AZ-104 Lab - Introduction to VMs in Azure

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group
2. Deploy a Linux virtual machine (VM) with default settings and configure a basic web server
3. Deploy a Windows virtual machine (VM) with default settings

---

## Prerequisites

* An active Azure subscription with permissions to create Resource Groups and VMs
* A user account that can log into the Azure Portal
* A local SSH key pair (if you do not already have one, refer to [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))
* A complex password that meets Azure’s requirements (for the Windows VM)

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value                                             |
| ----------------------------- | ------------------------------------------------- |
| **Region**                    | East US                                           |
| **Resource Group Name**       | az104-lab-rg                                      |
| **Linux VM Name**             | linux-webvm                                       |
| **Linux VM Admin Username**   | azureuser                                         |
| **SSH Public Key**            | Paste contents of `~/.ssh/id_rsa.pub`             |
| **Linux VM Image**            | Ubuntu Server 22.04 LTS (UbuntuLTS)               |
| **Linux VM Size**             | Standard\_DS1\_v2                                 |
| **Windows VM Name**           | windows-vm                                        |
| **Windows VM Admin Username** | azureadmin                                        |
| **Windows VM Admin Password** | (choose a complex password, e.g. `P@ssw0rd1234!`) |
| **Windows VM Image**          | Windows Server 2022 Datacenter                    |
| **Windows VM Size**           | Standard\_DS1\_v2                                 |

*Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric.*

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. In the left-hand menu, select **Resource groups**.
3. Click **+ Create**.
4. Under **Basics**, fill in:

   * **Subscription**: Your Azure subscription
   * **Resource group**: az104-lab-rg
   * **Region**: East US
5. Leave **Tags** blank.
6. Click **Review + create**, then **Create**.
7. Wait until you see “Deployment succeeded.”

---

## Task 2: Deploy Linux VM (UbuntuLTS)

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.
2. Click **+ Create** and choose **Azure virtual machine**.
3. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: az104-lab-rg
   * **Virtual machine name**: linux-webvm
   * **Region**: East US
   * **Image**: Ubuntu Server 22.04 LTS (UbuntuLTS)
   * **Size**: Standard\_DS1\_v2 (click **Change size**, select Standard\_DS1\_v2, then **Select**)
   * **Authentication type**: SSH public key
   * **Username**: azureuser
   * **SSH public key source**: Use existing key stored on your local machine
   * **SSH public key**: Paste contents of `~/.ssh/id_rsa.pub`
4. Under **Inbound ports**, ensure **SSH (22)** is selected.
5. Click **Next: Disks**, leave defaults (Premium SSD), then **Next: Networking**.
6. Under **Networking**:

   * **Virtual network**: Accept the automatically created linux-webvm-vnet
   * **Subnet**: Accept default (10.0.0.0/24)
   * **Public IP**: Accept the new linux-webvm-ip
   * **NIC network security group**: Select **Basic** and verify SSH (22) is allowed
   * Click **Add inbound port rule** and add:

     * **Service**: HTTP
     * **Destination port ranges**: 80
     * **Name**: Allow-HTTP-80
7. Click **Next: Management**, leave defaults, then **Next: Advanced**.
8. Under **Advanced**, leave defaults (no extensions, no custom data).
9. Click **Next: Tags**, leave blank, then **Next: Review + create**.
10. Review all settings (ensure they match the variables above), then **Create**.
11. Wait for “Deployment succeeded.”

---

## Task 3: Configure Linux VM as Web Server

1. In the Azure Portal, go to **Virtual machines**, then select **linux-webvm**.
2. On the **Overview** page, copy the **Public IP address** (e.g., 20.x.x.x).
3. Open your local terminal (or Azure Cloud Shell) and connect via SSH:

   ```
   ssh azureuser@<linux-webvm-public-ip>
   ```
4. Once logged in **on the VM**, perform these steps (capture evidence with screenshots):

   * Update package index (e.g., `sudo apt update`).
   * Install Apache2 (`sudo apt install -y apache2`).
   * Enable and start the Apache2 service (`sudo systemctl enable apache2 && sudo systemctl start apache2`).
5. In your browser, navigate to `http://<linux-webvm-public-ip>` and confirm you see the default Apache page. Screenshot this page.

---

## Task 4: Deploy Windows VM (Windows Server 2022 Datacenter)

1. In the Azure Portal, click **Virtual machines**.
2. Click **+ Create** and choose **Azure virtual machine**.
3. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: az104-lab-rg
   * **Virtual machine name**: windows-vm
   * **Region**: East US
   * **Image**: Windows Server 2022 Datacenter
   * **Size**: Standard\_DS1\_v2 (click **Change size**, select Standard\_DS1\_v2, then **Select**)
   * **Authentication type**: Password
   * **Username**: azureadmin
   * **Password**: (enter your complex password, e.g., P\@ssw0rd1234!)
4. Under **Inbound ports**, select **RDP (3389)**.
5. Click **Next: Disks**, leave defaults (Premium SSD), then **Next: Networking**.
6. Under **Networking**:

   * **Virtual network**: Accept the automatically created windows-vm-vnet
   * **Subnet**: Accept default (10.1.0.0/24)
   * **Public IP**: Accept the new windows-vm-ip
   * **NIC network security group**: Select **Basic** and verify RDP (3389) is allowed
7. Click **Next: Management**, leave defaults, then **Next: Advanced**.
8. Under **Advanced**, leave defaults.
9. Click **Next: Tags**, leave blank, then **Next: Review + create**.
10. Review all settings, then **Create**.
11. Wait for “Deployment succeeded.”

---

## Task 5: Verify Windows VM via RDP

1. In the Azure Portal, go to **Virtual machines** and select **windows-vm**.
2. On the **Overview** page, copy the **Public IP address** (e.g., 20.x.x.x).
3. Open your local RDP client and connect to that IP:

   * **Username**: azureadmin
   * **Password**: (the password you chose)
4. After logging in successfully, take a screenshot of the Windows Server desktop.

---

## Deliverables

1. **Resource Group Screenshot**

   * Show `az104-lab-rg` in **East US** with provisioning state “Succeeded.”

2. **Linux VM Verification**

   * SSH session screenshot showing Apache2 installed and running (`systemctl status apache2`).
   * Browser screenshot of `http://<linux-webvm-public-ip>` displaying the Apache default page.

3. **Windows VM Verification**

   * RDP session screenshot showing that you are logged into `windows-vm`.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-lab-rg**.
3. Click **Delete resource group**.
4. Type `az104-lab-rg` to confirm, then click **Delete**.

> This action permanently deletes both VMs, associated disks, NICs, public IPs, and network resources.

---

## Lab Tips

* Copy and paste the exact values for names, region, VM sizes, usernames, and images—any deviation will affect grading.
* When creating the Linux VM, opening port 80 under Networking (inbound rule) is mandatory for HTTP access.
* When creating the Windows VM, ensure port 3389 is allowed before attempting RDP.
* If you experience connection timeouts, verify the public IP and security rules in the Network Security Group (NSG).
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.

Good luck!
