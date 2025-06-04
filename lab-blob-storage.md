
# AZ-104 Lab Instructions: Azure Blob Storage Static Website Hosting & Large File Upload (GUI)

In this lab, you will use the Azure Portal (GUI) (and Azure Cloud Shell only if needed to retrieve SAS tokens) to complete the following tasks:

1. Create a new Resource Group  
2. Create a Storage Account and enable Static Website Hosting  
3. Configure index and error documents  
4. Upload a simple static website (HTML, CSS, JavaScript) to the `$web` container  
5. Upload a large file (e.g., a 100 MB video or archive) to a Blob container using the Azure Portal GUI  
6. Verify the static website is accessible and the large file can be downloaded  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups and Storage Accounts  
- A user account that can log into the Azure Portal  
- A basic static website package (e.g., `index.html`, `styles.css`, `404.html`) ready to upload  
- A large file (~100 MB or larger) on your local workstation  

---

## Lab Variables (Use Exactly These Values)

| Variable                         | Value                                    |
|----------------------------------|------------------------------------------|
| **Region**                       | East US                                  |
| **Resource Group Name**          | az104-blob-rg                            |
| **Storage Account Name**         | az104blobstatic (must be globally unique)|
| **Storage SKU**                  | Standard_LRS                             |
| **Static Website Index Document**| `index.html`                             |
| **Static Website Error Document**| `404.html`                               |
| **Local Static Site Folder**     | `./website-content` (path on your workstation) |
| **Static Site Files**            | `index.html`, `styles.css`, `404.html`   |
| **Large File Path (Local)**      | `./large-file.bin` (e.g., 100 MB file)   |
| **Blob Container for Large File**| `large-files`                            |

_Do not change any of these values. If you modify names, regions, or prefixes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-blob-rg`  
   - **Region**: **East US**  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until **“Deployment succeeded”** appears.

---

## Task 2: Create Storage Account and Enable Static Website

1. In the Azure Portal, click **Create a resource**, search for **Storage account**, then click **Create**.  
2. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-blob-rg**  
   - **Storage account name**: `az104blobstatic` (unique)  
   - **Region**: **East US**  
   - **Performance**: **Standard**  
   - **Replication**: **Locally-redundant storage (LRS)**  
3. Click **Next: Advanced**, leave defaults.  
4. Click **Next: Networking**, leave **Connectivity method** = **All networks** (public access).  
5. Click **Next: Data protection**, leave defaults.  
6. Click **Next: Encryption**, leave defaults.  
7. Click **Next: Tags**, leave blank.  
8. Click **Next: Review + create**, verify settings, then click **Create**.  
9. Wait for **“Deployment succeeded”**.  
10. Navigate to **Resource groups** → **az104-blob-rg** → **az104blobstatic**.  
11. In the storage account blade, scroll down and click **Static website** under **Data management**.  
12. Click **Enabled**.  
13. Under **Index document name**, enter `index.html`.  
14. Under **Error document path**, enter `404.html`.  
15. Click **Save**.  
16. Note the **Primary endpoint** URL (e.g., `https://az104blobstatic.z13.web.core.windows.net/`).  

   > _Take a screenshot of the **Static website** pane showing Enabled, Index document name, Error document path, and the Primary endpoint._

---

## Task 3: Upload Static Website Files to the `$web` Container

1. In the storage account blade (`az104blobstatic`), click **Containers** under **Data storage**.  
2. Click the container named `$web`.  
3. Click **Upload**.  
4. In the **Upload blob** pane:  
   - Click **Browse for files** and navigate to your local website folder (`./website-content`).  
   - Select `index.html`, `styles.css`, and `404.html`.  
   - Ensure **Overwrite if files exist** is **Checked**.  
5. Click **Upload**.  
6. Wait until all files show **Success**.  
7. Verify the files appear in the `$web` container.  

   > _Take a screenshot of the `$web` container showing `index.html`, `styles.css`, and `404.html`._

---

## Task 4: Verify Static Website Hosting

1. Open a new browser tab and navigate to the **Primary endpoint** (e.g., `https://az104blobstatic.z13.web.core.windows.net/`).  
2. Confirm the static website loads `index.html` correctly, including styling from `styles.css`.  
3. Navigate to a non-existent page (e.g., `https://az104blobstatic.z13.web.core.windows.net/nonexistent.html`).  
4. Confirm you see the content of `404.html`.  

   > _Take two screenshots: one showing the homepage, and one showing the 404 error page._

---

## Task 5: Create a Blob Container for Large Files

1. In the storage account blade, click **Containers**.  
2. Click **+ Container**.  
3. Under **New container**, configure:  
   - **Name**: `large-files`  
   - **Public access level**: **Private (no anonymous access)**  
4. Click **Create**.  
5. Verify `large-files` appears in the container list.  

   > _Take a screenshot of the container list showing `large-files`._

---

## Task 6: Upload a Large File Using the Azure Portal GUI

1. Ensure your large file (e.g., `large-file.bin`) is accessible on your local workstation.  
2. In the storage account blade, click **Containers** → select **large-files**.  
3. Click **Upload**.  
4. In the **Upload blob** pane:  
   - Click **Browse for files** and navigate to your local large file (e.g., `./large-file.bin`).  
   - Optionally adjust **Blob type** (default Block blob) and **Tier** (default Hot).  
   - Ensure **Overwrite if files exist** is **Checked** (if re-uploading).  
5. Click **Upload**.  
6. Wait until the portal shows **Success** for the upload—this may take several minutes for a 100 MB file.  
7. Verify `large-file.bin` appears in the `large-files` container.  

   > _Take a screenshot of the `large-files` container showing `large-file.bin`._

---

## Task 7: Generate a SAS URL to Download the Large File

1. In the storage account blade, click **Containers** → **large-files** → select `large-file.bin`.  
2. In the **Blob properties** pane, click **Generate SAS**.  
3. Under **Permissions**, check **Read**.  
4. Set an **Expiry** time (e.g., 1 hour from now).  
5. Click **Generate SAS token and URL**.  
6. Copy the **URL** (it includes the SAS token).  

   > _Take a screenshot of the **SAS token** pane showing the generated URL (you may redact part of the token)._

---

## Task 8: Verify Large File Download

1. On your local workstation, open a terminal or browser.  
2. Navigate to the SAS URL you copied (e.g., paste into a browser or use `curl`). Example using `curl`:  
   ```bash
   curl -O "https://az104blobstatic.blob.core.windows.net/large-files/large-file.bin?<sas_token>"
   ```

3. Confirm you can download `large-file.bin` completely (\~100 MB).

   * In the terminal, verify the file size matches the original.

   * In a browser, confirm the download completes.

   > *Take a screenshot of the terminal or browser showing the successful download.*

---

## Deliverables

1. **Resource Group Screenshot**

   * Show **`az104-blob-rg`** in **East US** with provisioning state **“Succeeded.”**

2. **Storage Account & Static Website Configuration Screenshots**

   * **az104blobstatic Overview** showing **Resource group**, **Region**, **Performance/Replication**.
   * **Static website** pane showing **Enabled**, **Index document name = index.html**, **Error document path = 404.html**, and **Primary endpoint**.

3. **\$web Container Screenshot**

   * Show contents of `$web` with `index.html`, `styles.css`, and `404.html`.

4. **Static Site Verification Screenshots**

   * Browser screenshot showing the static website homepage.
   * Browser screenshot showing the 404 error page.

5. **Container List Screenshot**

   * Show **Containers** list including `$web` and `large-files`.

6. **Large File Upload Screenshot**

   * Show `large-files` container with `large-file.bin` after uploading via the portal.

7. **SAS Token & URL Screenshot**

   * Show the generated SAS URL for downloading `large-file.bin` (you may redact part of the token).

8. **Large File Download Verification Screenshot**

   * Show terminal or browser confirming download of `large-file.bin` via the SAS URL.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.
2. Select **az104-blob-rg**.
3. Click **Delete resource group**.
4. Type **`az104-blob-rg`** to confirm, then click **Delete**.

> This action permanently deletes the Storage Account, File Share, VMs (if any), VNet, NSG, and any other resources under **`az104-blob-rg`**.

---

## Lab Tips

* Copy and paste the exact variable values for names, region, and SKU—any deviation will affect grading.
* For static website hosting, ensure your files are in the `$web` container root (no subfolders unless you adjust paths).
* The Azure Portal GUI can handle large file uploads, but be patient—100 MB may take a few minutes.
* When generating a SAS URL, grant only **Read** permission with a short expiry.
* Use a local terminal or browser to verify the large file download; check file size to confirm completeness.
* All configuration (including large file upload) must be performed through the Azure Portal GUI.

Good luck completing the lab tasks!
