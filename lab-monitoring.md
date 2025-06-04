
# AZ-104 Lab Instructions: Deploy Grafana on a Linux VM and Configure Azure AD Authentication

In this lab, you will use the Azure Portal (GUI) (and SSH into the VM) to complete the following tasks:

1. Create a Resource Group  
2. Deploy a Linux VM, install Grafana, and open required ports  
3. Register an Azure AD application to obtain Client ID and Client Secret  
4. Configure Grafana to use Azure AD for authentication  
5. Verify you can log into Grafana with an Azure AD account  

_Do not use Azure CLI or PowerShell for Azure resource creation—use the Azure Portal interface. For Grafana installation and configuration, SSH into the VM._

---

## Prerequisites

- An Azure subscription with permissions to create Resource Groups, Virtual Machines, and register Azure AD applications  
- A user account in your Azure AD tenant with Global Administrator or Application Administrator rights  
- A local SSH key pair (public key) ready for VM access  
- A Windows, macOS, or Linux workstation with an SSH client and a text editor  

---

## Lab Variables (Use Exactly These Values)

| Variable                          | Value / Notes                                                           |
|-----------------------------------|-------------------------------------------------------------------------|
| **Resource Group Name**           | `az104-grafana-rg`                                                      |
| **Region**                        | `East US`                                                               |
| **Linux VM Name**                 | `grafana-vm`                                                            |
| **Linux VM Admin Username**       | `azureuser`                                                             |
| **SSH Public Key**                | Paste contents of your local `~/.ssh/id_rsa.pub`                        |
| **Linux VM Image**                | `Ubuntu Server 22.04 LTS (UbuntuLTS)`                                    |
| **Linux VM Size**                 | `Standard_DS1_v2`                                                        |
| **NSG Name**                      | `grafana-nsg`                                                           |
| **Grafana Port**                  | `3000` (default HTTP)                                                   |
| **Azure AD App Name**             | `Grafana-Auth-App`                                                      |
| **Redirect URI**                  | `http://<grafana-vm-public-ip>:3000/login/azuread`                       |
| **Azure AD Tenant ID**            | (obtain from Azure AD overview)                                          |
| **Azure AD Client ID**            | (will be generated)                                                      |
| **Azure AD Client Secret**        | (will be generated)                                                      |

_Do not change any of these values. If you modify names, ports, or URIs, your lab validation will not match._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-grafana-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until **“Deployment succeeded”** appears.  
   - _Screenshot:_ Show **`az104-grafana-rg`** in **East US** with provisioning state **Succeeded**.

---

## Task 2: Deploy a Linux VM and Open Grafana Port

### 2.1 Create Network Security Group and Inbound Rule

1. In the Azure Portal, click **Create a resource**, search for **Network security group**, and click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-grafana-rg`  
   - **Name**: `grafana-nsg`  
   - **Region**: `East US`  
3. Click **Review + create**, then **Create**.  
4. Wait for **“Deployment succeeded”**.  
5. Navigate to **az104-grafana-rg** → **grafana-nsg** → **Inbound security rules** → **+ Add**.  
6. Create a rule to allow HTTP on port 3000:  
   - **Source**: `Any`  
   - **Source port ranges**: `*`  
   - **Destination**: `Any`  
   - **Destination port ranges**: `3000`  
   - **Protocol**: `TCP`  
   - **Action**: `Allow`  
   - **Priority**: `100`  
   - **Name**: `Allow-Grafana-3000`  
7. Click **Add**.  
   - _Screenshot:_ Show `Allow-Grafana-3000` in inbound rules.

### 2.2 Deploy the Linux VM

1. In the Azure Portal, click **Create a resource** → **Virtual machine**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: `az104-grafana-rg`  
   - **Virtual machine name**: `grafana-vm`  
   - **Region**: `East US`  
   - **Image**: `Ubuntu Server 22.04 LTS (UbuntuLTS)`  
   - **Size**: Click **Change size**, select **Standard_DS1_v2**, click **Select**  
   - **Authentication type**: **SSH public key**  
   - **Username**: `azureuser`  
   - **SSH public key source**: **Use existing key**  
   - **SSH public key**: Paste `~/.ssh/id_rsa.pub`  
3. Under **Inbound ports**, select **Allow selected ports** → check **SSH (22)**.  
4. Click **Next: Disks**, leave defaults.  
5. Click **Next: Networking**, configure:  
   - **Network security group**: Select **grafana-nsg**  
   - **Public IP**: **Create new** → name `grafana-vm-pip` → **OK**  
   - **Subnet**, **Virtual network**: Accept defaults (auto-created)  
6. Click **Next: Management**, leave defaults.  
7. Click **Next: Advanced**, leave defaults.  
8. Click **Next: Tags**, leave blank.  
9. Click **Review + create**, verify all settings, then **Create**.  
10. Wait for **“Deployment succeeded”**.  
    - _Screenshot:_ Show **grafana-vm** Overview with Public IP and **grafana-nsg** under Networking.

---

## Task 3: Install Grafana on the Linux VM

1. In the Azure Portal, navigate to **Resource groups** → **az104-grafana-rg** → **grafana-vm**.  
2. On **grafana-vm**’s **Overview** page, copy the **Public IP address** (e.g., `20.x.x.x`).  
3. Open a terminal on your workstation and connect via SSH:  
   ```bash
   ssh azureuser@<grafana-vm-public-ip>
   ```

4. Once connected, run the following commands to install and start Grafana:

   ```bash
   # Update package index
   sudo apt update

   # Install prerequisites
   sudo apt install -y apt-transport-https software-properties-common wget

   # Add Grafana APT repository
   wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
   echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

   # Install Grafana
   sudo apt update
   sudo apt install -y grafana

   # Enable and start Grafana service
   sudo systemctl enable grafana-server
   sudo systemctl start grafana-server

   # Verify Grafana is running
   sudo systemctl status grafana-server
   ```

5. Confirm that Grafana is running (`active (running)`).

   * *Screenshot:* Show `systemctl status grafana-server` output.

6. Exit the SSH session.

---

## Task 4: Register an Azure AD Application for Grafana

1. In the Azure Portal, click **Azure Active Directory** → **App registrations** → **+ New registration**.
2. Under **Basics**, configure:

   * **Name**: `Grafana-Auth-App`
   * **Supported account types**: **Accounts in this organizational directory only**
   * **Redirect URI**:

     * **Type**: `Web`
     * **URI**: `http://<grafana-vm-public-ip>:3000/login/azuread`
3. Click **Register**.
4. After registration, on the **Overview** blade, copy the **Application (client) ID**.

   * *Screenshot:* Show **Client ID** on the App registration Overview.
5. In the left menu, click **Certificates & secrets** → **+ New client secret**.
6. Under **Add a client secret**, configure:

   * **Description**: `GrafanaSecret`
   * **Expires**: **6 months** (or your choice)
   * Click **Add**.
7. Copy the **Value** of the newly created secret (do not copy the Secret ID).

   * *Screenshot:* Show **Client secret value** (obscure part if desired).
8. In the left menu, click **API permissions** → **+ Add a permission** → **Microsoft Graph** → **Delegated permissions** →

   * Search for and select **openid**, **profile**, **email**, and **User.Read** → click **Add permissions**.
9. Click **Grant admin consent** → **Yes** to grant the permissions.

   * *Screenshot:* Show **API permissions** with status **Granted for <Tenant>**.

---

## Task 5: Configure Grafana to Use Azure AD for Authentication

1. SSH into **grafana-vm** again:

   ```bash
   ssh azureuser@<grafana-vm-public-ip>
   ```

2. Open the Grafana configuration file in a text editor:

   ```bash
   sudo nano /etc/grafana/grafana.ini
   ```

3. In `grafana.ini`, locate the `[auth.azuread]` section (it may be commented out). Modify/Add the following lines (uncomment if needed):

   ```ini
   [auth.azuread]
   enabled = true
   allow_sign_up = true
   client_id = <Application (client) ID from Task 4>
   client_secret = <Client secret value from Task 4>
   scopes = openid profile email User.Read
   auth_url = https://login.microsoftonline.com/<yourtenant>.onmicrosoft.com/oauth2/v2.0/authorize
   token_url = https://login.microsoftonline.com/<yourtenant>.onmicrosoft.com/oauth2/v2.0/token
   api_url = https://graph.microsoft.com/oidc/userinfo
   tls_skip_verify_insecure = false
   allowed_organizations = <yourtenant>.onmicrosoft.com
   ```

   * Replace `<Application (client) ID from Task 4>`
   * Replace `<Client secret value from Task 4>`
   * Replace `<yourtenant>.onmicrosoft.com` with your actual tenant domain.

4. Save and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X`).

5. Restart Grafana:

   ```bash
   sudo systemctl restart grafana-server
   ```

6. Confirm Grafana is running:

   ```bash
   sudo systemctl status grafana-server
   ```

   * *Screenshot:* Show Grafana service status as **active (running)**.

7. Exit the SSH session.

---

## Task 6: Verify Azure AD Authentication in Grafana

1. In your browser, navigate to `http://<grafana-vm-public-ip>:3000`.
2. You should see the Grafana login page with an **“Azure AD”** button or link.
3. Click **Sign in with Azure AD** (or similar).
4. You will be redirected to the Azure AD login page. Sign in with a valid Azure AD user (e.g., your lab Global Administrator account).
5. After successful authentication, you should be redirected back to Grafana and logged in.

   * *Screenshot:* Show Grafana’s home dashboard after login.
6. If you are prompted to create an organization or assign an Organization Admin role, follow the prompts.

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-grafana-rg`** in **East US** with provisioning state **Succeeded**.

2. **Network Security Group Rule Screenshot**

   * Show **Allow-Grafana-3000** in **grafana-nsg** inbound rules.

3. **VM Overview Screenshot**

   * Show **grafana-vm** Overview with Public IP and **grafana-nsg** under Networking.

4. **Grafana Service Screenshot**

   * SSH session showing `systemctl status grafana-server` as **active (running)**.

5. **Azure AD App Registration Screenshots**

   * **Overview** showing **Application (client) ID** for `Grafana-Auth-App`.
   * **Certificates & secrets** showing the **Client secret value**.
   * **API permissions** showing **openid**, **profile**, **email**, **User.Read** granted with admin consent.

6. **Grafana Configuration Screenshot**

   * SSH session showing the edited `[auth.azuread]` section in `/etc/grafana/grafana.ini`.

7. **Grafana Login Screenshot**

   * Browser screenshot showing **Sign in with Azure AD** option on Grafana login page.

8. **Grafana Authenticated Screenshot**

   * Browser screenshot showing Grafana’s welcome/dashboard after logging in via Azure AD.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the resources:

1. In the Azure Portal:

   * Delete **grafana-vm** (this removes the VM, disks, and NIC).
   * Delete **grafana-nsg**.
   * Delete **az104-grafana-rg** resource group (this also removes any dependent networking and public IP).
   * In **Azure AD** → **App registrations**, delete **Grafana-Auth-App**.

> This action ensures no lingering charges or resources remain.

---

## Lab Tips

* The **Redirect URI** must match exactly (including protocol, port, and path) between Azure AD app registration and Grafana configuration.
* If Grafana fails to start after editing `grafana.ini`, check the system logs:

  ```bash
  sudo journalctl -u grafana-server -f
  ```
* Ensure the VM’s NSG allows inbound traffic on port **3000**.
* When granting **API permissions**, always click **Grant admin consent** to avoid login errors.
* Use `nano` or any text editor available on the VM to modify configuration.

Good luck completing the lab tasks!
