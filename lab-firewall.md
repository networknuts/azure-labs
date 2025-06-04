
# AZ-104 Lab Instructions: Deploy Azure Firewall with a Private Linux VM and Configure DNAT & Filtering Rules

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create a Virtual Network with two subnets: one for the Azure Firewall and one for the Linux VM  
3. Deploy an Azure Firewall into the designated firewall subnet, assign a public IP, and configure:  
   - **DNAT rule** to forward SSH (TCP/22) from the firewall’s public IP to the private Linux VM  
   - **Network rule** to allow VM outbound traffic (except Facebook)  
   - **Application rule** to block access to **facebook.com**  
4. Deploy a Linux VM into the VM subnet with **no public IP**  
5. Configure an NSG on the VM subnet to permit SSH only from the firewall’s private IP  
6. Verify connectivity:  
   - SSH from your workstation to the firewall’s public IP and confirm you land on the Linux VM  
   - From the Linux VM, confirm you **cannot** access `facebook.com`  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface, except where instructed to SSH into the VM to validate rules._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, Virtual Networks, Subnets, Network Security Groups, Azure Firewall, and Virtual Machines  
- A user account that can log into the Azure Portal  
- A local SSH key pair (if you do not already have one, see [Generate SSH keys](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys))  

---

## Lab Variables (Use Exactly These Values)

| Variable                        | Value                              |
|---------------------------------|------------------------------------|
| **Region**                      | East US                            |
| **Resource Group Name**         | az104-fw-rg                        |
| **Virtual Network Name**        | az104-fw-vnet                      |
| **VNet Address Space**          | 10.100.0.0/16                      |
| **Firewall Subnet Name**        | AzureFirewallSubnet                |
| **Firewall Subnet Prefix**      | 10.100.0.0/24                      |
| **VM Subnet Name**              | VMSubnet                           |
| **VM Subnet Prefix**            | 10.100.1.0/24                      |
| **NSG Name (VM Subnet)**        | az104-fw-vm-nsg                    |
| **Azure Firewall Name**         | az104-firewall                     |
| **Firewall Public IP Name**     | az104-fw-pip                       |
| **DNAT Rule Name**              | ssh-dnat-rule                      |
| **DNAT Rule Public Port**       | 22                                 |
| **DNAT Rule Target Private IP** | 10.100.1.4 (Linux VM private IP)   |
| **DNAT Rule Target Port**       | 22                                 |
| **Network Rule Name**           | allow-vm-outbound                   |
| **Network Rule Source**         | 10.100.1.4 (Linux VM private IP)   |
| **Network Rule Destination**    | Any (0.0.0.0/0) except facebook.com |
| **Application Rule Name**       | block-facebook                     |
| **Application Rule Source**     | 10.100.1.4 (Linux VM private IP)   |
| **Application Rule Target FQDN**| `facebook.com`                     |
| **Linux VM Name**               | vm-private                         |
| **Linux VM Admin Username**     | azureuser                          |
| **SSH Public Key**              | Paste contents of `~/.ssh/id_rsa.pub` |
| **Linux VM Image**              | Ubuntu Server 22.04 LTS (UbuntuLTS)|
| **Linux VM Size**               | Standard_DS1_v2                    |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-fw-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.  

---

## Task 2: Create Virtual Network and Subnets

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for **Virtual network**, then click **Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-fw-rg**  
   - **Name**: `az104-fw-vnet`  
   - **Region**: `East US`  
4. Click **Next: IP Addresses**.  
5. Under **IP Addresses**, set:  
   - **IPv4 address space**: `10.100.0.0/16`  
6. Under **Subnets**, delete any default subnet entry. Then click **+ Add subnet** twice:  

   1. **First Subnet (Firewall)**:  
      - **Subnet name**: `AzureFirewallSubnet`  
      - **Subnet address range**: `10.100.0.0/24`  
      - Click **Add**.  

   2. **Second Subnet (VM)**:  
      - Click **+ Add subnet** again.  
      - **Subnet name**: `VMSubnet`  
      - **Subnet address range**: `10.100.1.0/24`  
      - Click **Add**.  

7. Click **Review + create**, verify **Address space** = `10.100.0.0/16` and that both subnets appear, then click **Create**.  
8. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Create Network Security Group (NSG) for VM Subnet

1. In the Azure Portal, click **Create a resource**, search for **Network security group**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-fw-rg**  
   - **Name**: `az104-fw-vm-nsg`  
   - **Region**: `East US`  
3. Click **Review + create**, then **Create**.  
4. Wait for **“Deployment succeeded”** before proceeding.  
5. Navigate to **Resource groups** → **az104-fw-rg** → **az104-fw-vm-nsg**.  
6. Under **Inbound security rules**, click **+ Add**.  
7. Configure the rule to **allow SSH from the Firewall subnet** only:  
   - **Source**: `IP Addresses`  
   - **Source IP addresses/CIDR ranges**: `10.100.0.0/24` (AzureFirewallSubnet)  
   - **Source port ranges**: `*`  
   - **Destination**: `Any`  
   - **Destination port ranges**: `22`  
   - **Protocol**: `TCP`  
   - **Action**: `Allow`  
   - **Priority**: `100`  
   - **Name**: `Allow-SSH-From-FW`  
8. Click **Add**.  
9. Under **Inbound security rules** again, add a second rule to **deny all other inbound** (default Azure behavior if no other allow). Ensure there is no rule permitting SSH from internet.  
10. Leave **Outbound security rules** at defaults (allow all).

---

## Task 4: Deploy Linux VM into VMSubnet (No Public IP)

1. In the Azure Portal, click **Virtual machines** in the left-hand menu.  
2. Click **+ Create** and choose **Azure virtual machine**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fw-rg`  
   - **Virtual machine name**: `vm-private`  
   - **Region**: `East US`  
   - **Availability options**: Leave as **No infrastructure redundancy required** (default)  
   - **Image**: **Ubuntu Server 22.04 LTS (UbuntuLTS)**  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, then **Select**  
   - **Authentication type**: SSH public key  
   - **Admin username**: `azureuser`  
   - **SSH public key source**: Use existing key stored on your local machine  
   - **SSH public key**: Paste the contents of `~/.ssh/id_rsa.pub`  
4. Under **Inbound ports**, set **Public inbound ports** to **None** (no Public IP).  
5. Click **Next: Disks**, leave defaults (Managed disk, Premium SSD).  
6. Click **Next: Networking**.  
7. Under **Networking**, configure:  
   - **Virtual network**: Select **az104-fw-vnet**  
   - **Subnet**: Select **VMSubnet**  
   - **Public IP**: **None**  
   - **Network security group**: **Basic** → Select **az104-fw-vm-nsg**  
8. Click **Next: Management**, leave defaults.  
9. Click **Next: Advanced**, leave defaults.  
10. Click **Next: Tags**, leave blank.  
11. Click **Next: Review + create**, verify all settings (ensure **No Public IP** and correct NSG), then **Create**.  
12. Wait for **“Deployment succeeded”**.  
13. Navigate to **Resource groups** → **az104-fw-rg** → **vm-private** and confirm:  
    - **No Public IP address**  
    - **Subnet = VMSubnet**  
    - **NSG = az104-fw-vm-nsg**  
    - Take a screenshot of the VM’s **Overview** page.

---

## Task 5: Deploy Azure Firewall with Public IP

1. In the Azure Portal, click **Create a resource**, search for **Firewall**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fw-rg`  
   - **Firewall name**: `az104-firewall`  
   - **Region**: `East US`  
3. Click **Next: Firewall policy**, leave default (no policy).  
4. Click **Next: Virtual networks**.  
5. Under **Virtual network**, select **az104-fw-vnet**.  
6. Under **Subnet**, ensure it shows **AzureFirewallSubnet** (mandatory).  
7. Under **Public IP address**, click **Create new**:  
   - **Name**: `az104-fw-pip`  
   - **SKU**: **Standard** (default)  
   - **Assignment**: **Static**  
   - Click **OK**.  
8. Click **Next: Tags**, leave blank, then **Next: Review + create**.  
9. Verify all settings (especially **Virtual network** and **Subnet** = `AzureFirewallSubnet`), then click **Create**.  
10. Wait until you see **“Deployment succeeded”**.  
11. Navigate to **Resource groups** → **az104-fw-rg** → **az104-firewall** to confirm:  
    - **Public IP** = `az104-fw-pip`  
    - **Subnet** = `AzureFirewallSubnet`  
    - Take a screenshot of the Firewall’s **Overview** page.  

---

## Task 6: Configure Azure Firewall Rules

### 6.1 DNAT Rule (Forward SSH to VM)

1. In the Azure Portal, navigate to **Resource groups** → **az104-fw-rg** → **az104-firewall**.  
2. In the left-hand menu, select **Rules** → **DNAT rules** → **+ Add**.  
3. Under **Basics**, configure:  
   - **Name**: `ssh-dnat-rule`  
   - **Priority**: `100`  
   - **Rule action**: **Dnat**  
4. Under **Source**:  
   - **Source addresses**: `0.0.0.0/0` (allow SSH from anywhere)  
   - **Protocol**: `TCP`  
   - **Source ports**: `*`  
5. Under **Destination**:  
   - **Destination addresses**: Enter the firewall’s public IP address (select `az104-fw-pip`)  
   - **Destination ports**: `22`  
6. Under **Translated address**:  
   - **Translated address**: Enter `10.100.1.4` (private IP of `vm-private`)  
   - **Translated port**: `22`  
7. Click **Add**.  
8. Confirm the DNAT rule appears in the list with **Priority = 100**.

### 6.2 Network Rule (Allow VM Outbound Except Facebook)

1. In **Rules** → **Network rules** → **+ Add**.  
2. Under **Basics**, configure:  
   - **Name**: `allow-vm-outbound`  
   - **Priority**: `200`  
   - **Rule action**: **Allow**  
3. Under **Source**:  
   - **Source type**: `IP Addresses`  
   - **Source IP addresses**: `10.100.1.4` (private IP of `vm-private`)  
4. Under **Protocol**:  
   - **Protocol**: `Any` (to allow all protocols except blocked by application rule)  
5. Under **Destination**:  
   - **Destination type**: `Service Tag`  
   - **Service Tag**: `Internet` (allow all internet traffic)  
   - **Destination ports**: `*`  
6. Click **Add**.  
7. Confirm the network rule appears in the list with **Priority = 200**.

### 6.3 Application Rule (Block facebook.com)

1. In **Rules** → **Application rules** → **+ Add**.  
2. Under **Basics**, configure:  
   - **Name**: `block-facebook`  
   - **Priority**: `300`  
   - **Rule action**: **Deny**  
3. Under **Source**:  
   - **Source type**: `IP Addresses`  
   - **Source IP addresses**: `10.100.1.4` (private IP of `vm-private`)  
4. Under **Target**:  
   - **Protocol**: `HTTP`  
   - **Port**: `80`  
   - **Target FQDNs**: Enter `facebook.com` (exact match)  
5. Click **Add**.  
6. Confirm the application rule appears in the list with **Priority = 300**.

---

## Task 7: Configure UDR (Optional for Forcing Firewall as Default Route)

> **Note:** By default, Azure Firewall will NAT and route traffic without a User-Defined Route if both network rule and application rule are defined. If outbound traffic does not flow through the firewall automatically, you may need to create a UDR (route table) on `VMSubnet` pointing 0.0.0.0/0 to the firewall private IP.

1. In the Azure Portal, click **Create a resource**, search for **Route table**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-fw-rg`  
   - **Name**: `az104-fw-rt`  
   - **Region**: `East US`  
3. Click **Review + create**, then **Create**.  
4. After creation, navigate to **Resource groups** → **az104-fw-rg** → **az104-fw-rt**.  
5. Under **Settings**, click **Routes** → **+ Add**.  
6. Configure:  
   - **Route name**: `default-route`  
   - **Address prefix**: `0.0.0.0/0`  
   - **Next hop type**: `Virtual Appliance`  
   - **Next hop address**: Enter `10.100.0.4` (the private IP of the Azure Firewall’s internal NIC)  
   - Click **Add**.  
7. Under **Settings**, click **Subnets** → **+ Associate**.  
8. Select **az104-fw-vnet** → **VMSubnet** → **Associate**.  
9. Confirm the route table is associated with **VMSubnet**.

---

## Task 8: Verify SSH Connectivity via NSG and Firewall

1. From your local workstation, open an SSH session targeting the firewall’s public IP:  
   ```bash
   ssh azureuser@<firewall-public-ip>
   ```

2. When prompted, the firewall DNAT rule should forward you to `vm-private`’s SSH.
3. Inside the SSH session, you should see the shell prompt of `vm-private`.

   * Take a screenshot showing your prompt on `vm-private`.
4. Exit the SSH session to return to your workstation.

---

## Task 9: Verify Facebook Block from VM

1. SSH back into the Linux VM (via firewall) using:

   ```bash
   ssh azureuser@<firewall-public-ip>
   ```
2. Once on `vm-private`, attempt to reach Facebook:

   ```bash
   curl -I http://facebook.com
   ```
3. You should see that the connection is **blocked** or times out (no HTTP response).

   * Take a screenshot of the failed attempt, showing a timeout or connection refused.
4. Test outbound DNS or HTTP to another site (e.g., `curl -I http://example.com`) to confirm outbound traffic still works.

   * Take a screenshot showing a successful response from `example.com`.
5. Exit the SSH session.

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-fw-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **VNet & Subnets Screenshot**

   * Show **`az104-fw-vnet`** with two subnets:

     * `AzureFirewallSubnet (10.100.0.0/24)`
     * `VMSubnet (10.100.1.0/24)`

3. **NSG Screenshot**

   * Show **`az104-fw-vm-nsg`** inbound rules, including **Allow-SSH-From-FW**.

4. **VM Overview Screenshot**

   * Show **`vm-private`** in **East US**, **Subnet = VMSubnet**, **No Public IP**, and **NSG = az104-fw-vm-nsg**.

5. **Firewall Overview Screenshot**

   * Show **`az104-firewall`** in **East US**, **Subnet = AzureFirewallSubnet**, **Public IP = az104-fw-pip**.

6. **Firewall Rule Screenshots**

   * **DNAT Rules**: Show **`ssh-dnat-rule`** with priority 100 forwarding port 22 to `10.100.1.4:22`.
   * **Network Rules**: Show **`allow-vm-outbound`** permitting `10.100.1.4` to Internet.
   * **Application Rules**: Show **`block-facebook`** denying `facebook.com` for `10.100.1.4`.

7. **UDR & Association Screenshot** (if UDR is configured)

   * Show **`az104-fw-rt`** route with **0.0.0.0/0 → 10.100.0.4** and association to **VMSubnet**.

8. **SSH Connection Verification Screenshot**

   * Show an SSH prompt on **`vm-private`** after connecting to `<firewall-public-ip>`.

9. **Facebook Block Verification Screenshot**

   * Show `curl -I http://facebook.com` failing (timeout or connection refused).
   * Show `curl -I http://example.com` succeeding (HTTP 200 OK).

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-fw-rg**.
3. Click **Delete resource group**.
4. Type **`az104-fw-rg`** to confirm, then click **Delete**.

> This action permanently deletes the VNet, subnets, NSG, VM, Firewall, Public IP, route table, and any other resources under **`az104-fw-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, address spaces, prefixes, and rule names—any deviation will affect grading.
* Ensure the **AzureFirewallSubnet** is named exactly (must be `/FirewallSubnet` for Azure Firewall).
* When creating the DNAT rule, double-check that the **Destination address** is the firewall’s public IP and that the **Translated address** is the VM’s private IP.
* The NSG on **VMSubnet** must allow SSH only from **AzureFirewallSubnet (10.100.0.0/24)**.
* If the VM cannot access the internet at all, ensure the **Network rule** and (if used) UDR are properly configured.
* To verify the Facebook block, use `curl` (not `ping`) since the application rule blocks HTTP (DNS or TCP).
* All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell for resource creation or management.

Good luck completing the lab tasks!
