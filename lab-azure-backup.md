
# AZ-104 Lab Instructions: Azure Backup for Cloud Linux VM and On-Prem Windows VM

In this lab, you will set up and test Azure Backup for two scenarios:

1. **Azure Cloud VM (RHEL 9)**  
2. **On-Premises Windows VM (Windows Server 2019 Datacenter)**  

For each VM, you will:
- Configure backup (using Recovery Services vault)
- Take an initial backup
- Delete or modify a test file on the VM
- Restore the file from backup

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface for the Azure VM and through the MARS agent UI for the on-prem Windows VM._

---

## Prerequisites

- An Azure subscription with Owner or Backup Contributor access  
- A running RHEL 9 VM in Azure (e.g., Resource Group `az104-backup-rg`, VM name `linux-rhel9-vm`)  
- A Windows Server 2019 Datacenter VM running on-premises (e.g., VM name `local-win2019`)  
- Administrator credentials for both VMs  
- Internet access from the on-prem Windows VM to connect to Azure Backup  

---

## Lab Variables (Use Exactly These Values)

| Scenario                        | Variable                              | Value                               |
|---------------------------------|---------------------------------------|-------------------------------------|
| **Common**                      | Resource Group                        | `az104-backup-rg`                   |
| **Recovery Services Vault**     | Vault Name                            | `az104-rsv`                         |
| **Vault Region**                |                                       | `East US`                           |
| **Vault Storage Redundancy**    |                                       | `Locally-redundant (LRS)`           |
| **Azure Linux VM**              | VM Name                               | `linux-rhel9-vm`                    |
| **Azure Linux VM Admin User**   |                                       | (as configured when VM was created) |
| **On-Prem Windows VM**          | VM Name                               | `local-win2019`                     |
| **On-Prem Windows Admin User**  |                                       | (e.g., `Administrator`)             |
| **Protected Item**              | Backup Item Name (Linux)              | `linux-rhel9-vm`                    |
| **Protected Item**              | Backup Item Name (Windows)            | `local-win2019`                     |
| **Test File (Linux)**           | Path                                  | `/home/azureuser/testfile.txt`      |
| **Test File (Windows)**         | Path                                  | `C:\BackupTest\testfile.txt`        |
| **MARS Agent Download URL**     |                                       | https://aka.ms/azuresitemover       |

_Do not change any of these values. Your lab validation depends on these names._

---

## Task 1: Create a Recovery Services Vault

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**, then open **az104-backup-rg**.  
3. Click **+ Create** → search for **Recovery Services vault** → click **Create**.  
4. Under **Basics**, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-backup-rg`  
   - **Vault name**: `az104-rsv`  
   - **Region**: `East US`  
5. Click **Review + create** → **Create**.  
6. Wait until **“Deployment succeeded”** appears.  
7. Navigate to **az104-rsv** (Recovery Services vault blade) and confirm:  
   - **Properties** → **Backup Configuration** shows default: **Storage redundancy = LRS**.  
   - Screenshot the vault’s **Overview** for validation.

---

## Task 2: Enable Backup for Azure Linux VM (RHEL 9)

1. In the Azure Portal, go to **az104-rsv** → **Backup** under **Getting Started**.  
2. Under **Where is your workload running?**, select **Azure**, and under **What do you want to back up?**, choose **Azure Virtual Machine**.  
3. Click **Prepare Infrastructure** (this step registers the vault if needed).  
4. Under **Select items to back up**, click **+ Add**. A list of VMs in `az104-backup-rg` appears.  
5. Select **linux-rhel9-vm** → click **OK**.  
6. Under **Backup policy**, verify **DefaultPolicy** (daily at 12:00 AM, retention 30 days) is selected.  
7. Click **Enable backup**.  
8. In **Backup Items** → **Azure Virtual Machine**, confirm **linux-rhel9-vm** appears with **Protected** = Yes.  
   - Screenshot the protected items list showing `linux-rhel9-vm`.

---

## Task 3: Take an Initial Backup of Azure Linux VM

1. In **az104-rsv** → **Backup items** → **Azure Virtual Machine**, click on **linux-rhel9-vm**.  
2. On the VM’s backup blade, click **Backup now**.  
3. In the **Backup Now** pane:  
   - **Retention range**: leave default (e.g., 30 days)  
   - Click **OK**.  
4. Wait until the backup job completes:  
   - Navigate to **az104-rsv** → **Backup jobs** → **Completed jobs**.  
   - Confirm one entry shows **Status = Completed**, with item = `linux-rhel9-vm`.  
   - Screenshot the job list.  

---

## Task 4: Delete a Test File on Linux VM and Restore from Backup

1. SSH into **linux-rhel9-vm** via Cloud Shell or your own SSH client:  
   ```bash
   ssh azureuser@<linux-rhel9-vm-public-IP>
   ```

2. On the VM, create a test file:

   ```bash
   echo "Azure Backup Test" | sudo tee /home/azureuser/testfile.txt
   ls /home/azureuser/testfile.txt
   ```

   * Screenshot the existence of `/home/azureuser/testfile.txt`.
3. Delete the test file:

   ```bash
   sudo rm /home/azureuser/testfile.txt
   ls /home/azureuser/testfile.txt
   ```

   * Confirm the file is gone (no output).
   * Screenshot the deletion.
4. In the Azure Portal, navigate to **az104-rsv** → **Backup items** → **Azure Virtual Machine** → **linux-rhel9-vm** → **Restore VM**.
5. In the **Restore Virtual Machine** pane:

   * Under **Restore point type**, choose **Latest**.
   * Under **Restore point**, leave default.
   * Under **Recovered VM name**, enter `linux-rhel9-vm-restored`.
   * Under **Restore configuration**, select **Restore disks** (we will mount the disk locally).
   * Click **OK**.
6. Wait until **Restore jobs** → **Completed jobs** shows **VM restored**.

   * Screenshot the completed restore job.
7. Navigate to **Resource groups** → **az104-backup-rg** → **linux-rhel9-vm-restored**.
8. Stop the original `linux-rhel9-vm` to avoid IP conflicts.
9. On `linux-rhel9-vm-restored`, detach the OS disk and attach to a “helper” Linux VM in the same VNet if desired—or simply start the restored VM (it will get a new public IP).
10. Once `linux-rhel9-vm-restored` is running, SSH into it and verify `/home/azureuser/testfile.txt` exists:

    ```bash
    ssh azureuser@<linux-rhel9-vm-restored-public-IP>
    ls /home/azureuser/testfile.txt
    cat /home/azureuser/testfile.txt
    ```

    * Screenshot the restored file and its contents.
11. Delete `linux-rhel9-vm-restored` (or deallocate) when done to avoid extra costs.

---

## Task 5: Configure Backup for On-Prem Windows VM (Windows Server 2019)

> **Note:** For on-premises machines, Azure Backup uses the MARS (Microsoft Azure Recovery Services) agent.

### 5.1: Create and Configure a Protection Policy for On-Prem VM

1. In the Azure Portal, open **az104-rsv** → **Backup**.
2. Under **Where is your workload running?**, select **On-premises** and for **What do you want to back up?**, choose **Files and folders** (initially; we’ll install MARS to back up system-state or volumes).
3. Click **Prepare Infrastructure** → This downloads the MARS agent installer.

   * Under **Download** section, click **Download Windows Server backup agent** → save `AzureRecoveryServicesBackupInstaller.exe` locally.
   * Copy the **Vault credentials** file (.vaultcredentials) for registration.
   * Screenshot the download instructions.

### 5.2: Install MARS Agent on the On-Prem Windows VM

1. Log in to **local-win2019** as Administrator.
2. Copy the MARS installer (`AzureRecoveryServicesBackupInstaller.exe`) and the vault credentials file to **C:\Temp** (or any local folder).
3. Run `AzureRecoveryServicesBackupInstaller.exe`.
4. In the **Microsoft Azure Recovery Services Agent Setup** wizard:

   * Click **Next** → Accept license → click **Next**.
   * For **Vault Credentials**, browse to the downloaded `.vaultcredentials` file and click **Next**.
   * The agent installs and registers the server with **az104-rsv**.
   * When prompted to configure Microsoft Update, select **Install updates** (optional).
   * Click **Finish** when done.
5. In Windows, open **Microsoft Azure Backup** from the Start Menu.

   * Verify the server is registered to **az104-rsv** (the console shows vault name).
   * Screenshot the agent UI showing registration.

### 5.3: Schedule a Backup of a Test File or Folder

1. In **Microsoft Azure Backup** on the Windows VM, click **Schedule Backup**.
2. In the **Getting Started** wizard:

   * Under **Items to Back Up**: click **Add Items** → select a folder (create `C:\BackupTest` first).
   * Create and add a test file:

     * In File Explorer, create `C:\BackupTest\testfile.txt` with some content (e.g., “Azure Backup Test”).
   * In the wizard, browse to `C:\BackupTest` and click **OK**.
3. Under **Backup Schedule**:

   * Choose **Daily** at a time (e.g., 7:00 PM).
   * Click **Next**.
4. Under **Retention Policy**, keep defaults (retain daily backups for 30 days).

   * Click **Next**.
5. Under **Vault Storage Settings**, leave default.

   * Click **Next** → **Finish**.
6. In the **Microsoft Azure Backup** console, click **Backup Now**.
7. In the **Backup Now** dialog, choose **Full** and click **OK**.
8. Wait until the job completes (monitor the **Jobs** tab).

   * Screenshot the successful backup job.

---

## Task 6: Delete Test File on Windows VM and Restore from Backup

1. In Windows on **local-win2019**, open File Explorer → navigate to `C:\BackupTest` → delete `testfile.txt`.

   * Confirm the file is gone.
   * Screenshot the empty folder.
2. In **Microsoft Azure Backup** console, click **Recover Data**.
3. In the **Recovery Wizard**:

   * Under **Select Recovery Type**, choose **Files and Folders** → click **Next**.
   * Under **Select Recovery Point**, choose the **Latest** recovery point → click **Next**.
   * Under **Select Items to Recover**, navigate to `C:\BackupTest\testfile.txt`, check the box → click **Next**.
   * Under **Destination for Recovery**, choose **Alternate location** (e.g., `C:\RestoreTest`).
   * Create folder `C:\RestoreTest` if needed; set that folder → click **Next** → **Recover**.
4. Wait until recovery completes (monitor the **Jobs** tab).

   * Screenshot the job completion status.
5. In File Explorer, navigate to `C:\RestoreTest` → verify `testfile.txt` appears and open it.

   * Screenshot the restored file and contents.

---

## Deliverables

1. **Recovery Services Vault Screenshot**

   * Show **`az104-rsv`** in **East US**, **Storage redundancy = LRS**.

2. **Azure Linux VM Backup Screenshots**

   * **Backup Items** showing `linux-rhel9-vm` as Protected.
   * **Backup jobs** showing the initial backup completed.
   * SSH session showing creation and deletion of `/home/azureuser/testfile.txt`.
   * **Restore jobs** showing VM restore completed.
   * SSH session on `linux-rhel9-vm-restored` showing `/home/azureuser/testfile.txt` restored.

3. **MARS Agent Installation & Registration Screenshots**

   * MARS agent installer wizard (vault credentials step).
   * **Microsoft Azure Backup** console showing `local-win2019` registered to **az104-rsv**.

4. **On-Prem Windows VM Backup Screenshots**

   * **Schedule Backup** wizard showing `C:\BackupTest` selected.
   * **Jobs** tab showing initial backup completed.
   * File Explorer screenshot of `C:\BackupTest\testfile.txt` before deletion.

5. **On-Prem Windows VM Recovery Screenshots**

   * **Recover Data** wizard showing recovery point and item selected.
   * **Jobs** tab showing recovery completed.
   * File Explorer screenshot of `C:\RestoreTest\testfile.txt` restored and contents.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete test resources:

1. In Azure Portal:

   * Delete `linux-rhel9-vm-restored` VM.
   * In **az104-rsv** → **Backup items**, stop protection for `linux-rhel9-vm` and **Delete Backup Data**.
   * In **az104-rsv** → **Backup items**, under **File Recovery**, remove the on-prem registration.
   * Delete Recovery Services vault `az104-rsv`.
   * Delete Resource Group `az104-backup-rg` (this also deletes any restored VM resources).

2. On-Prem Windows VM (`local-win2019`):

   * Uninstall **Microsoft Azure Recovery Services Agent** from **Control Panel** → **Programs and Features**.
   * Delete `C:\BackupTest` and `C:\RestoreTest` folders.

> **Note:** Unregistering the server and cleaning up prevents additional charges.

---

## Lab Tips

* For the Azure VM restore, choosing **“Restore disks”** allows you to inspect a mounted disk before rotating on production; for simplicity, we restored as a new VM.
* When creating the test file on Linux, ensure the path `/home/azureuser/testfile.txt` matches the Admin user’s home directory.
* The MARS agent will only back up files/folders you explicitly select; system-state or application backup requires a different approach.
* To view detailed logs in the MARS agent, click **Actions** → **View Logs** in the agent console.
* Make sure the on-premises VM can communicate outbound to Azure Backup endpoints (port 443).
* All configuration steps for the Azure VM backup must be done through the Recovery Services vault blade in the Azure Portal.

Good luck completing the lab tasks!
