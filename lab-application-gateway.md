
# AZ-104 Lab Instructions: Deploy Application Gateway with Round-Robin Across Two Regions

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create three Virtual Networks:  
   - **AppGateway-VNet** (for the Application Gateway)  
   - **VM1-VNet** (for `vm1`, in Region A)  
   - **VM2-VNet** (for `vm2`, in Region B)  
3. Configure VNet peering:  
   - **Local peering** between **AppGateway-VNet** and **VM1-VNet** (same region)  
   - **Global peering** between **AppGateway-VNet** and **VM2-VNet** (different region)  
4. Deploy one Linux VM in **VM1-VNet** and another in **VM2-VNet**, each running Apache (`httpd`) with distinct `index.html` content  
5. Create an Application Gateway in **AppGateway-VNet**, add both VMs as backend targets, and configure a listener, HTTP settings, and routing rule to achieve round-robin distribution  
6. Verify that browsing the Application Gateway’s public IP alternates between the two VM homepages  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface, except where instructed to SSH into VMs to configure Apache._

---

## Prerequisites

- Active Azure subscription with permission to create Resource Groups, Virtual Networks, VNet Peerings (local & global), VMs, and an Application Gateway  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                             | Value                                      |
|--------------------------------------|--------------------------------------------|
| **Region A (AppGW & VM1)**           | East US                                    |
| **Region B (VM2)**                   | West Europe                                |
| **Resource Group Name**              | az104-agw-rg                               |
| **AppGateway-VNet Name**             | vnet-appgw                                 |
| **AppGateway-VNet Address Space**    | 10.10.0.0/16                               |
| **AppGateway-Subnet Name**           | subnet-appgw                               |
| **AppGateway-Subnet Prefix**         | 10.10.0.0/24                               |
| **VM1-VNet Name**                    | vnet-vm1                                   |
| **VM1-VNet Address Space**           | 10.20.0.0/16                               |
| **VM1-Subnet Name**                  | subnet-vm1                                 |
| **VM1-Subnet Prefix**                | 10.20.1.0/24                               |
| **VM2-VNet Name**                    | vnet-vm2                                   |
| **VM2-VNet Address Space**           | 10.30.0.0/16                               |
| **VM2-Subnet Name**                  | subnet-vm2                                 |
| **VM2-Subnet Prefix**                | 10.30.1.0/24                               |
| **Peering Name (AppGW→VM1)**         | peer-appgw-to-vm1                          |
| **Peering Name (VM1→AppGW)**         | peer-vm1-to-appgw                          |
| **Peering Name (AppGW→VM2)**         | peer-appgw-to-vm2                          |
| **Peering Name (VM2→AppGW)**         | peer-vm2-to-appgw                          |
| **Linux VM 1 Name**                  | vm1                                        |
| **Linux VM 2 Name**                  | vm2                                        |
| **VM Admin Username**                | azureuser                                  |
| **SSH Public Key**                   | Paste contents of `~/.ssh/id_rsa.pub`      |
| **Linux VM Image**                   | Ubuntu Server 22.04 LTS (UbuntuLTS)        |
| **Linux VM Size**                    | Standard_DS1_v2                            |
| **App Gateway Name**                 | agw-frontend                               |
| **App Gateway Tier/Size**            | Standard_v2 (default)                      |
| **AppGW Frontend IP Name**           | agw-public-ip                              |
| **AppGW Frontend IP SKU/Assignment** | Standard / Static                           |
| **AppGW Listener Name**              | http-listener                              |
| **AppGW Backend Pool Name**          | pool-vms                                   |
| **AppGW HTTP Settings Name**         | http-settings                              |
| **AppGW Routing Rule Name**          | rule-http                                  |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-agw-rg`  
   - **Region**: **East US** (Region A)  
5. Leave **Tags** blank.  
6. Click **Review + create**, then click **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Create Virtual Networks and Subnets

### 2.1 Create AppGateway-VNet (East US)

1. In the Azure Portal, click **Create a resource**.  
2. Search for **Virtual network**, then click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-agw-rg`  
   - **Name**: `vnet-appgw`  
   - **Region**: **East US**  
4. Click **Next: IP Addresses**.  
5. Under **IP Addresses**, configure:  
   - **IPv4 address space**: `10.10.0.0/16`  
6. Under **Subnets**, delete any default subnet (if present) then click **+ Add subnet**:  
   - **Subnet name**: `subnet-appgw`  
   - **Subnet address range**: `10.10.0.0/24`  
   - Click **Add**.  
7. Click **Review + create**, verify **Address space** and **Subnet** match the variables, then click **Create**.  
8. Wait for **“Deployment succeeded”**.

---

### 2.2 Create VM1-VNet (East US)

1. In the Azure Portal, click **Create a resource**, search for **Virtual network**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-agw-rg`  
   - **Name**: `vnet-vm1`  
   - **Region**: **East US** (same as AppGW)  
3. Click **Next: IP Addresses**.  
4. Under **IP Addresses**, configure:  
   - **IPv4 address space**: `10.20.0.0/16`  
5. Under **Subnets**, delete any default, then click **+ Add subnet**:  
   - **Subnet name**: `subnet-vm1`  
   - **Subnet address range**: `10.20.1.0/24`  
   - Click **Add**.  
6. Click **Review + create**, verify settings, then click **Create**.  
7. Wait for **“Deployment succeeded”**.

---

### 2.3 Create VM2-VNet (West Europe)

1. In the Azure Portal, click **Create a resource**, search for **Virtual network**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-agw-rg`  
   - **Name**: `vnet-vm2`  
   - **Region**: **West Europe** (Region B)  
3. Click **Next: IP Addresses**.  
4. Under **IP Addresses**, configure:  
   - **IPv4 address space**: `10.30.0.0/16`  
5. Under **Subnets**, delete any default, then click **+ Add subnet**:  
   - **Subnet name**: `subnet-vm2`  
   - **Subnet address range**: `10.30.1.0/24`  
   - Click **Add**.  
6. Click **Review + create**, verify settings, then click **Create**.  
7. Wait for **“Deployment succeeded”**.

---

## Task 3: Configure VNet Peering

### 3.1 Peer AppGateway-VNet ↔︎ VM1-VNet (Local Peering, same region)

1. In the Azure Portal, navigate to **Resource groups** → **az104-agw-rg** → **vnet-appgw**.  
2. In the left-hand menu of **vnet-appgw**, select **Peerings** and click **+ Add**.  
3. Under **Basics**, configure:  
   - **Peering link name (this virtual network)**: `peer-appgw-to-vm1`  
   - **Subscription**: Same subscription  
   - **Virtual network (remote)**: Select **vnet-vm1**  
   - **Peering link name (remote virtual network)**: Enter `peer-vm1-to-appgw`  
4. Under **Configuration**, ensure:  
   - **Allow virtual network access**: **Enabled**  
   - **Allow forwarded traffic** / **Allow gateway transit**: leave defaults (**Disabled**)  
5. Click **Add**.  
6. Wait for **“Peering created”**.  

7. Still in **vnet-appgw → Peerings**, confirm **peer-appgw-to-vm1** is **Connected**.  
8. Navigate to **Resource groups** → **az104-agw-rg** → **vnet-vm1** → **Peerings**.  
9. Confirm **peer-vm1-to-appgw** exists with **Connected** status.  

---

### 3.2 Peer AppGateway-VNet ↔︎ VM2-VNet (Global Peering, different regions)

1. In the Azure Portal, navigate to **Resource groups** → **az104-agw-rg** → **vnet-appgw**.  
2. In the left-hand menu of **vnet-appgw**, select **Peerings** and click **+ Add**.  
3. Under **Basics**, configure:  
   - **Peering link name (this virtual network)**: `peer-appgw-to-vm2`  
   - **Subscription**: Same subscription  
   - **Virtual network (remote)**: Select **vnet-vm2** (in West Europe)  
   - **Peering link name (remote virtual network)**: Enter `peer-vm2-to-appgw`  
4. Under **Configuration**, ensure:  
   - **Allow virtual network access**: **Enabled**  
   - **Allow forwarded traffic**: leave **Disabled**  
   - **Allow gateway transit**: **Disabled**  
5. Click **Add**.  
6. Wait for **“Peering created”**.  

7. Still in **vnet-appgw → Peerings**, confirm **peer-appgw-to-vm2** is **Connected**.  
8. Navigate to **Resource groups** → **az104-agw-rg** → **vnet-vm2** → **Peerings**.  
9. Confirm **peer-vm2-to-appgw** exists with **Connected** status.  

---

## Task 4: Deploy Linux VMs and Configure Apache

### Common Steps for Both VMs

1. In the Azure Portal, click **Virtual machines** → **+ Create** → **Azure virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-agw-rg`  
   - **Virtual machine name**:  
     - `vm1` (for Region A, East US)  
     - `vm2` (for Region B, West Europe)  
   - **Region**:  
     - For `vm1`: **East US**  
     - For `vm2`: **West Europe**  
   - **Availability options**: Leave as **No infrastructure redundancy required**  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: **Standard_DS1_v2**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, set **Public inbound ports** to **None** (no Public IP).  
4. Click **Next: Disks**, leave defaults.  
5. Click **Next: Networking**.  

   - For `vm1`:  
     - **Virtual network**: `vnet-vm1`  
     - **Subnet**: `subnet-vm1`  
     - **Public IP**: **None**  
     - **Network security group**: **Basic** → Leave default (SSH allowed from Azure )  
   - For `vm2`:  
     - **Virtual network**: `vnet-vm2`  
     - **Subnet**: `subnet-vm2`  
     - **Public IP**: **None**  
     - **Network security group**: **Basic** → Leave default  

6. Click **Next: Management**, leave defaults.  
7. Click **Next: Advanced**, leave defaults.  
8. Click **Next: Tags**, leave blank.  
9. Click **Next: Review + create**, verify all settings, then **Create**.  
10. Wait for **“Deployment succeeded”** for both VMs.  
11. For each VM, navigate to **Resource groups** → **az104-agw-rg** → **vm1** (or `vm2`) and confirm:  
    - **No Public IP address**  
    - **Correct Virtual network & Subnet**  
    - Take a screenshot of each VM’s **Overview** page.

---

### Configure Apache on VM1 and VM2

#### 4.1 SSH into VM1 and Configure Apache

1. In the Azure Portal, navigate to **Resource groups** → **az104-agw-rg** → **vm1**.  
2. On **vm1**’s **Overview** page, click **Connect** → **Bastion** (or use Cloud Shell’s SSH).  
3. In the Bastion pane (or SSH session), authenticate with:  
   - **Username**: `azureuser`  
   - **SSH private key**: your private key (`~/.ssh/id_rsa`)  
4. Once connected, run the following commands to install Apache (`httpd`) and customize the homepage:  
   ```bash
   sudo apt update
   sudo apt install -y apache2
   echo "<html><body><h1>VM1: Hello from East US</h1></body></html>" | sudo tee /var/www/html/index.html
   sudo systemctl enable apache2
   sudo systemctl start apache2
  ```

5. Verify Apache is running:

   ```bash
   sudo systemctl status apache2
   ```

   Take a screenshot showing **“active (running)”**.
6. Exit the SSH session for `vm1`.

#### 4.2 SSH into VM2 and Configure Apache

1. In the Azure Portal, navigate to **Resource groups** → **az104-agw-rg** → **vm2**.
2. On **vm2**’s **Overview** page, click **Connect** → **Bastion** (or use Cloud Shell’s SSH).
3. In the Bastion pane (or SSH session), authenticate with:

   * **Username**: `azureuser`
   * **SSH private key**: your private key (`~/.ssh/id_rsa`)
4. Once connected, run the following commands to install Apache (`httpd`) and customize the homepage:

   ```bash
   sudo apt update
   sudo apt install -y apache2
   echo "<html><body><h1>VM2: Hello from West Europe</h1></body></html>" | sudo tee /var/www/html/index.html
   sudo systemctl enable apache2
   sudo systemctl start apache2
   ```
5. Verify Apache is running:

   ```bash
   sudo systemctl status apache2
   ```

   Take a screenshot showing **“active (running)”**.
6. Exit the SSH session for `vm2`.

---

## Task 5: Create a Public IP for the Application Gateway

1. In the Azure Portal, click **Create a resource**, search for **Public IP address**, and click **Create**.
2. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: `az104-agw-rg`
   * **Name**: `agw-public-ip`
   * **Region**: **East US**
   * **SKU**: **Standard**
   * **IP version**: **IPv4**
   * **Assignment**: **Static**
3. Click **Review + create**, then click **Create**.
4. Wait for **“Deployment succeeded”**.
5. Navigate to **Resource groups** → **az104-agw-rg** → **agw-public-ip**, take note of the **IP address**, and screenshot the **Overview** blade.

---

## Task 6: Deploy Application Gateway and Configure Round Robin

1. In the Azure Portal, click **Create a resource**, search for **Application Gateway**, and click **Create**.
2. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: `az104-agw-rg`
   * **Region**: **East US** (where `vnet-appgw` resides)
   * **Name**: `agw-frontend`
   * **Tier**: **Standard\_v2** (default)
   * **Autoscale**: **Disabled** (we’ll use default instance count)
3. Click **Next: Frontends**.
4. Under **Frontend IP configuration**, configure:

   * **Type**: **Public**
   * **Name**: `agw-public-ip`
   * **Public IP address**: Select **agw-public-ip**
   * Click **Add**
5. Click **Next: Backends**.
6. Under **Backend pools**, click **Add a backend pool**:

   * **Name**: `pool-vms`
   * **Target type**: **IP address or FQDN**
   * **IP addresses**: Enter the private IP of `vm1` (e.g., `10.20.1.4`) and the private IP of `vm2` (e.g., `10.30.1.4`)—retrieve each from its VM’s **Overview** page.
   * Click **Add**
7. Click **Next: Routing rules**.
8. Under **HTTP settings**, click **Add HTTP setting**:

   * **Name**: `http-settings`
   * **Backend protocol**: **HTTP**
   * **Backend port**: `80`
   * **Pick host name from backend address**: Leave **Disabled**
   * **Path-based routing**: Leave **No**
   * **Affinitized cookie-based affinity**: Leave **Disabled**
   * Click **Add**
9. Still under **Routing rules**, click **Add a rule**:

   * **Name**: `rule-http`
   * **Rule type**: **Basic**
   * **Listener**: Click **Add a listener**, then:

     * **Name**: `http-listener`
     * **Frontend IP**: Select `agw-public-ip`
     * **Frontend port**: **80** (default)
     * **Protocol**: **HTTP**
     * Click **Add**
   * **Backend targets**: Select **pool-vms**
   * **HTTP setting**: Select **http-settings**
   * Click **Add**
10. Click **Next: Tags**, leave blank, then **Next: Review + create**.
11. Verify all settings (especially frontend IP, backend pool IP addresses, and HTTP settings), then click **Create**.
12. Wait for **“Deployment succeeded”**.
13. Navigate to **Resource groups** → **az104-agw-rg** → **agw-frontend** and confirm:

    * **Frontend IP configuration** = `agw-public-ip`
    * **Backend pool** = `pool-vms` (showing both VM IPs)
    * **HTTP settings** = `http-settings`
    * **Routing rule** = `rule-http`
    * Take a screenshot of the **Overview** blade and **Backend pools** blade showing both VMs.

---

## Task 7: Verify Round-Robin Distribution via Application Gateway

1. Copy the **Frontend public IP** (e.g., `20.x.x.x`) from the **agw-public-ip** resource.
2. From your local workstation or Cloud Shell, run:

   ```bash
   curl http://<appgw-public-ip>/
   ```

   or open `http://<appgw-public-ip>/` in a browser.
3. Observe the response:

   * One request should return:

     ```
     <h1>VM1: Hello from East US</h1>
     ```
   * The next request (refresh) should return:

     ```
     <h1>VM2: Hello from West Europe</h1>
     ```
   * Successive requests should alternate between the two VM homepages (round-robin).
4. Take screenshots of at least three consecutive requests showing alternating content (e.g., VM1 → VM2 → VM1).

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-agw-rg`** in the appropriate region with provisioning state **“Succeeded.”**

2. **VNet Overview Screenshots**

   * **vnet-appgw** (East US) with **Address space = 10.10.0.0/16**, **Subnet = 10.10.0.0/24**
   * **vnet-vm1** (East US) with **Address space = 10.20.0.0/16**, **Subnet = 10.20.1.0/24**
   * **vnet-vm2** (West Europe) with **Address space = 10.30.0.0/16**, **Subnet = 10.30.1.0/24**

3. **VNet Peering Screenshots**

   * **vnet-appgw → Peerings** showing **peer-appgw-to-vm1** with **Connected** status
   * **vnet-vm1 → Peerings** showing **peer-vm1-to-appgw** with **Connected** status
   * **vnet-appgw → Peerings** showing **peer-appgw-to-vm2** with **Connected** status
   * **vnet-vm2 → Peerings** showing **peer-vm2-to-appgw** with **Connected** status

4. **VM Overview Screenshots**

   * **vm1**: Show **No Public IP**, **Subnet = subnet-vm1**, **Region = East US**
   * **vm2**: Show **No Public IP**, **Subnet = subnet-vm2**, **Region = West Europe**

5. **Apache Status Screenshots**

   * On **vm1**, show `systemctl status apache2` running
   * On **vm2**, show `systemctl status apache2` running

6. **Application Gateway Screenshots**

   * **agw-frontend → Overview** showing **Frontend IP = agw-public-ip**, **Backend Pool = pool-vms**, **HTTP settings = http-settings**, **Routing rule = rule-http**
   * **agw-frontend → Backend pools** showing both VM private IP addresses registered

7. **Round-Robin Verification Screenshots**

   * At least three consecutive `curl http://<appgw-public-ip>/` outputs (or browser refreshes), showing alternating content from **VM1** and **VM2**

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-agw-rg**.
3. Click **Delete resource group**.
4. Type **`az104-agw-rg`** to confirm, then click **Delete**.

> This action permanently deletes all VNets, peerings, VMs, and the Application Gateway along with its Public IP.

---

## Lab Tips

* Copy and paste the exact variable values for names, regions, address spaces, subnets, peerings, and Application Gateway components—any deviation will affect grading.
* For VM private IPs, retrieve them from each VM’s **Overview** page under **Networking**.
* Ensure local peering is used for VNets in the same region (vnet-appgw ↔︎ vnet-vm1) and global peering for VNets in different regions (vnet-appgw ↔︎ vnet-vm2).
* When configuring the Application Gateway backend pool, use the VMs’ **private IP addresses**, not hostnames.
* Round-robin behavior can be verified by running `curl` multiple times or refreshing the browser window to see alternating responses.
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation or management.

Good luck completing the lab tasks!
