
# AZ-104 Lab Instructions: Create Load Balancer with Two Linux VMs and Verify Round Robin

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Deploy two Linux virtual machines (VMs) with Apache HTTP Server installed  
3. Customize each VM’s `index.html` so they return different content  
4. Create an Azure Load Balancer, add both VMs to its backend pool, and configure a health probe and load‐balancing rule for HTTP (port 80)  
5. Verify round‐robin distribution by repeatedly connecting to the Load Balancer’s frontend IP and observing alternating content  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface, except where instructed to SSH into the VMs to configure Apache._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Virtual Networks, Virtual Machines, Load Balancers, and Public IPs  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  
- Basic familiarity with SSHing into a Linux VM and installing Apache HTTP Server (`httpd` or `apache2`)  

---

## Lab Variables (Use Exactly These Values)

| Variable                        | Value                          |
|---------------------------------|--------------------------------|
| **Region**                      | East US                        |
| **Resource Group Name**         | az104-lb-rg                    |
| **Virtual Network Name**        | az104-lb-vnet                  |
| **VNet Address Space**          | 10.0.0.0/16                    |
| **Subnet Name**                 | default                        |
| **Subnet Prefix**               | 10.0.0.0/24                    |
| **Network Security Group Name** | az104-lb-nsg                   |
| **NIC NSG Rule Name**           | Allow-HTTP-80                  |
| **Linux VM 1 Name**             | lb-vm-1                        |
| **Linux VM 2 Name**             | lb-vm-2                        |
| **Admin Username**              | azureuser                      |
| **SSH Public Key**              | Paste contents of `~/.ssh/id_rsa.pub` |
| **Linux VM Image**              | Ubuntu Server 22.04 LTS (UbuntuLTS) |
| **Linux VM Size**               | Standard_DS1_v2                |
| **Public IP Name (LB)**         | lb-public-ip                   |
| **Load Balancer Name**          | az104-lb                       |
| **Frontend IP Configuration**   | lb-frontend-ip                 |
| **Backend Pool Name**           | lb-backend-pool                |
| **Health Probe Name**           | http-probe                     |
| **Load Balancing Rule Name**    | http-rule                      |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group and Virtual Network

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-lb-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then click **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.  

8. In the Azure Portal, click **Create a resource** (upper-left corner).  
9. Search for **Virtual network**, then click **Create**.  
10. Under **Basics**, configure:  
    - **Subscription**: Your subscription  
    - **Resource group**: `az104-lb-rg`  
    - **Name**: `az104-lb-vnet`  
    - **Region**: `East US`  
11. Click **Next: IP Addresses**.  
12. Under **IP Addresses**, set:  
    - **IPv4 address space**: `10.0.0.0/16`  
13. Under **Subnets**, ensure there is a single default subnet with:  
    - **Subnet name**: `default`  
    - **Subnet address range**: `10.0.0.0/24`  
14. Click **Review + create**, verify settings, then click **Create**.  
15. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 2: Create Network Security Group and Inbound Rule

1. In the Azure Portal, click **Create a resource**, search for **Network security group**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-lb-rg`  
   - **Name**: `az104-lb-nsg`  
   - **Region**: `East US`  
3. Click **Review + create**, then **Create**.  
4. Wait for **“Deployment succeeded”** before proceeding.  
5. Navigate to **Resource groups** → **az104-lb-rg** → **az104-lb-nsg**.  
6. Under **Inbound security rules**, click **+ Add**.  
7. Configure the rule as follows to allow HTTP on port 80 from anywhere within Azure (or the internet):  
   - **Source**: `Any`  
   - **Source port ranges**: `*`  
   - **Destination**: `Any`  
   - **Destination port ranges**: `80`  
   - **Protocol**: `TCP`  
   - **Action**: `Allow`  
   - **Priority**: `100`  
   - **Name**: `Allow-HTTP-80`  
8. Click **Add**.  
9. Leave **Outbound security rules** at their defaults.

---

## Task 3: Deploy Two Linux VMs (lb-vm-1 and lb-vm-2)

You will create two identical Ubuntu VMs in the same vNet and subnet, each with no public IP. Afterwards, you will SSH into each VM to install Apache and customize the `index.html`.

### VM Deployment Steps (applies to both lb-vm-1 and lb-vm-2)

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** → **Azure virtual machine**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-lb-rg`  
   - **Virtual machine name**:  
     - For the first VM, enter `lb-vm-1`  
     - For the second VM, repeat steps and enter `lb-vm-2`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required** (default)  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of your `~/.ssh/id_rsa.pub` file  
4. Under **Inbound ports**, set **Public inbound ports** to **None**.  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD).  
6. Click **Next: Networking**.  
7. Under **Networking**, configure:  
   - **Virtual network**: Select **az104-lb-vnet**  
   - **Subnet**: Select `default`  
   - **Public IP**: **None** (no public IP)  
   - **Network security group**: Select **Basic** → Associate **az104-lb-nsg**  
   - **Accelerated networking**: Leave as **Off**  
8. Click **Next: Management**, leave defaults.  
9. Click **Next: Advanced**, leave defaults.  
10. Click **Next: Tags**, leave blank.  
11. Click **Next: Review + create**, verify all settings, then **Create**.  
12. Wait for **“Deployment succeeded”**.  
13. Repeat the entire process for `lb-vm-2`, ensuring you select the same vNet/subnet/NSG and no public IP.

---

## Task 4: Install Apache and Customize `index.html` on Each VM

Now that both VMs are running, you will SSH into each to install Apache and set a unique homepage.

1. In the Azure Portal, navigate to **Resource groups** → **az104-lb-rg** → **lb-vm-1**.  
2. On **lb-vm-1**’s **Overview** page, click **Connect** → **Bastion** (or use your preferred SSH method).  
3. In the Bastion pane:  
   - **Bastion host**: If you have an existing Bastion, select it; otherwise, use **SSH command** in Cloud Shell to connect.  
   - **Username**: `azureuser`  
   - **SSH private key**: Browse to your private key (`~/.ssh/id_rsa`) if prompted  
4. Once connected, run the following commands to install Apache and customize the homepage:  
   ```bash
   sudo apt update
   sudo apt install -y apache2
   echo "<html><body><h1>VM-1: Hello from lb-vm-1</h1></body></html>" | sudo tee /var/www/html/index.html
   sudo systemctl enable apache2
   sudo systemctl start apache2
   ```

5. Confirm Apache is running:

   ```bash
   sudo systemctl status apache2
   ```

   *Take a screenshot of the status showing “active (running)”.*

6. Exit the SSH session for `lb-vm-1`.

7. Repeat steps 1–6 for **lb-vm-2**, but modify the homepage to:

   ```bash
   echo "<html><body><h1>VM-2: Hello from lb-vm-2</h1></body></html>" | sudo tee /var/www/html/index.html
   ```

---

## Task 5: Create a Public IP for the Load Balancer

1. In the Azure Portal, click **Create a resource**, search for **Public IP address**, and click **Create**.
2. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: `az104-lb-rg`
   * **Name**: `lb-public-ip`
   * **Region**: `East US`
   * **SKU**: **Standard**
   * **IP version**: **IPv4**
   * **Assignment**: **Static**
3. Click **Review + create**, then click **Create**.
4. Wait for **“Deployment succeeded”**.
5. Navigate to **Resource groups** → **az104-lb-rg** → **lb-public-ip** and note the **IP address** or take a screenshot of the **Overview** blade.

---

## Task 6: Create Azure Load Balancer and Configure Frontend, Backend, and Health Probe

1. In the Azure Portal, click **Create a resource**, search for **Load Balancer**, and click **Create**.
2. Under **Basics**, configure:

   * **Subscription**: Your subscription
   * **Resource group**: Select **az104-lb-rg**
   * **Name**: `az104-lb`
   * **Region**: `East US`
   * **SKU**: **Standard**
   * **Type**: **Public**
3. Click **Next: Frontend IP configuration**.
4. Under **Frontend IP configuration**, configure:

   * **Name**: `lb-frontend-ip`
   * **IP version**: **IPv4**
   * **Public IP address**: Select **lb-public-ip**
5. Click **Next: Backend pools**.
6. Under **Backend pools**, click **Add a backend pool**:

   * **Name**: `lb-backend-pool`
   * **Virtual network**: Select **az104-lb-vnet**
   * **Association**: Add **lb-vm-1** and **lb-vm-2** network interfaces (select their NICs)
   * Click **Add**
7. Click **Next: Health probe**.
8. Under **Health probe**, click **Add a health probe**:

   * **Name**: `http-probe`
   * **Protocol**: **HTTP**
   * **Port**: `80`
   * **Path**: `/`
   * **Interval**: `15` seconds (default)
   * **Unhealthy threshold**: `2` (default)
   * Click **Add**
9. Click **Next: Load balancing rules**.
10. Under **Load balancing rules**, click **Add a rule**:

    * **Name**: `http-rule`
    * **IP version**: **IPv4**
    * **Frontend IP address**: Select `lb-frontend-ip`
    * **Protocol**: **TCP**
    * **Port**: `80`
    * **Backend port**: `80`
    * **Backend pool**: Select `lb-backend-pool`
    * **Health probe**: Select `http-probe`
    * **Idle timeout (minutes)**: `4` (default)
    * **Floating IP (direct server return)**: **Disabled** (default)
    * Click **Add**
11. Click **Next: Review + create**, verify all settings (frontend, backend pool, probe, rule), then click **Create**.
12. Wait for **“Deployment succeeded”**.
13. Navigate to **Resource groups** → **az104-lb-rg** → **az104-lb** and confirm in the **Overview**:

    * **Frontend IP configuration** = `lb-frontend-ip`
    * **Backend pool** = `lb-backend-pool` (showing two VMs)
    * **Health probe** = `http-probe`
    * **Load balancing rule** = `http-rule`

---

## Task 7: Verify Round Robin Distribution

1. Copy the **Frontend IP address** of `lb-frontend-ip` (e.g., `20.x.x.x`).
2. From your local workstation or Cloud Shell, open a web browser or use `curl`/`wget` to repeatedly request the homepage:

   ```bash
   curl http://<load-balancer-ip>/
   ```

   or simply enter `http://<load-balancer-ip>/` in your browser’s address bar.
3. Observe the response:

   * One request should return:

     ```
     <h1>VM-1: Hello from lb-vm-1</h1>
     ```
   * The next request (or refresh) should return:

     ```
     <h1>VM-2: Hello from lb-vm-2</h1>
     ```
   * Successive requests should alternate between the two.
4. Take screenshots illustrating at least three consecutive requests showing round‐robin behavior (e.g., VM-1 → VM-2 → VM-1).

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-lb-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **VM Overview Screenshots**

   * **lb-vm-1**: Show **No Public IP**, **Subnet = default**, and NSG = `az104-lb-nsg`.
   * **lb-vm-2**: Show **No Public IP**, **Subnet = default**, and NSG = `az104-lb-nsg`.

3. **Apache Status Screenshots**

   * On **lb-vm-1**, show `systemctl status apache2` (or `httpd`) running.
   * On **lb-vm-2**, show `systemctl status apache2` (or `httpd`) running.

4. **Load Balancer Configuration Screenshots**

   * **az104-lb** Overview blade showing **Frontend IP = lb-frontend-ip**, **Backend Pool = lb-backend-pool**, **Health Probe = http-probe**, **Rule = http-rule**.
   * **Backend Pool** pane showing both `lb-vm-1` and `lb-vm-2` NICs.
   * **Health Probes** pane showing `http-probe` and its status.

5. **Round-Robin Verification Screenshots**

   * At least three consecutive outputs from `curl http://<load-balancer-ip>/` (or browser refreshes), showing alternating content from **VM-1** and **VM-2**.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-lb-rg**.
3. Click **Delete resource group**.
4. Type **`az104-lb-rg`** to confirm, then click **Delete**.

> This action permanently deletes the VNet, NSG, both VMs, the Load Balancer, its Public IP, and any other resources under **`az104-lb-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, prefixes, usernames, image, and Load Balancer settings—any deviation will affect grading.
* Ensure each VM’s NSG allows inbound HTTP (port 80) so the Load Balancer health probe and traffic can reach them.
* Use Bastion or Cloud Shell to SSH into each VM to install Apache and customize the `index.html`.
* The Load Balancer health probe must succeed on port 80 for each VM before traffic is forwarded.
* Round‐robin behavior can be verified by clearing browser cache or using `curl -s` to force fresh requests.
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation or management.

Good luck completing the lab tasks!


