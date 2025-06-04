# AZ-104 Lab Instructions: Using Azure Storage Explorer on Windows

In this lab, you will install Azure Storage Explorer on your Windows laptop, then use it to connect to:

1. An individual Blob container  
2. An individual File share  
3. Your entire Azure subscription  

You will perform basic operations (upload, download, delete) on blobs and files.

---

## Prerequisites

- A Windows 10 or later laptop with internet access  
- An Azure subscription with at least one Storage Account  
  - **Resource group:** `az104-se-rg`  
  - **Storage account name:** `az104seaccount` (replace with your actual account)  
- A local file (e.g., `testfile.txt`) available on your laptop for upload  

---

## Task 1: Install Azure Storage Explorer

1. Open your browser and navigate to the [Azure Storage Explorer download page](https://azure.microsoft.com/features/storage-explorer/).  
2. Click **Download** for the Windows installer (MSI).  
3. Run the downloaded `.msi` file and follow the prompts:  
   - Accept the license agreement  
   - Click **Next** → **Install**  
   - When installation completes, click **Finish**  

4. Launch **Azure Storage Explorer** from the Start Menu (search for “Storage Explorer”).  
5. Confirm that the application opens without errors.

---

## Task 2: Connect to Your Azure Subscription (Full Account Access)

1. In Azure Storage Explorer, click the **plug icon** ( Connect ) in the left-hand toolbar.  
2. Select **Add an Azure Account** and click **Next**.  
3. A browser window will open—sign in with your Azure credentials.  
4. After successful authentication, return to Storage Explorer and confirm you see your subscription listed under **Subscriptions**.  
   - Expand **Subscriptions** → **(Your Tenant)** → **(Your Subscription Name)**  
   - Under your subscription, you should see all resource groups, including `az104-se-rg`.  

> _Screenshot: Show the expanded subscription tree with your resource group visible._

---

## Task 3: Connect to an Individual Blob Container Using Connection String

> _This method is useful if you only want access to a single Blob container, not your entire subscription._

1. In the Azure Portal, navigate to **Resource groups** → **az104-se-rg** → **az104seaccount**.  
2. In the storage account blade, click **Access keys** under **Security + networking**.  
3. Copy **Connection string (key1)** to your clipboard.  
4. In Storage Explorer, click **Connect** → **Use a connection string** → **Next**.  
5. Paste the connection string into the **Connection string** field, then click **Next** → **Connect**.  
6. Expand the newly connected **Storage Accounts** node in the left pane; you will see `az104seaccount-connection`.  
7. Expand **Blob Containers**, then expand `blob-container-1` (replace with your actual container).  

> _Screenshot: Show the connected storage account under “Storage Accounts (connected via connection string).”_

---

## Task 4: Connect to an Individual File Share Using SAS URL

> _Use this method if you want access only to a single File share._

1. In the Azure Portal, navigate to **Resource groups** → **az104-se-rg** → **az104seaccount**.  
2. Click **File shares** under **Data storage**, then select your file share (e.g., `fileshare1`).  
3. Click **Generate SAS** on the top bar. In the SAS pane:  
   - Under **Allowed permissions**, check **Read**, **Write**, **Delete**, **List**.  
   - Set an **Expiry** (e.g., 2 hours from now).  
   - Click **Generate SAS token and URL**.  
4. Copy the **URL** field (it includes the SAS token).  
5. In Storage Explorer, click **Connect** → **Use a SAS URI** → **Next**.  
6. Paste the SAS URL into the **SAS URI** field, then click **Next** → **Connect**.  
7. Expand the new **File Shares** node under “SAS-connected resources” and confirm you see `fileshare1`.  

> _Screenshot: Show the connected file share under “File Shares (connected via SAS).”_

---

## Task 5: Upload and Delete a Blob in a Blob Container

1. In Storage Explorer, under **Storage Accounts (connected via Connection string)** → **az104seaccount-connection** → **Blob Containers**, click your blob container (e.g., `blob-container-1`).  
2. Right-click **blob-container-1** and select **Open**. The right pane shows existing blobs (if any).  
3. Click the **Upload** button on the toolbar.  
   - In the **Upload Files** dialog, click **Browse** and navigate to a local file (e.g., `testfile.txt`).  
   - Click **Upload**.  
4. Wait until you see `testfile.txt` in the container.  
   - Confirm its size matches your local file.  

   > _Screenshot: Show `testfile.txt` listed in the blob container._

5. To delete the blob:  
   - Right-click `testfile.txt` → **Delete** → **Yes**.  
   - Confirm `testfile.txt` is removed from the list.  

   > _Screenshot: Show the container list after deletion (no `testfile.txt`)._

---

## Task 6: Upload and Delete a File in a File Share

1. In Storage Explorer, under **File Shares (connected via SAS)** → **fileshare1**, double-click to open. You’ll see the root directory contents.  
2. Click **Upload** → **Upload Files**.  
   - Browse to a local file (e.g., `testfile.txt`) and click **Open**.  
   - Click **Upload**.  
3. Wait until you see `testfile.txt` appear in the file share root.  

   > _Screenshot: Show `testfile.txt` listed in the file share._

4. To delete the file:  
   - Right-click `testfile.txt` → **Delete** → **Yes**.  
   - Confirm `testfile.txt` is removed from the file share.  

   > _Screenshot: Show the file share list after deletion (no `testfile.txt`)._

---

## Task 7: Connect to Your Whole Azure Account Using Azure AD

1. In Storage Explorer, click **Connect** → **Add an Azure Account** → **Next**.  
2. If you’re already signed in, Storage Explorer may refresh and display your subscription under **Subscriptions**. If not, sign in with your Azure AD credentials.  
3. Expand **Subscriptions** → your **Tenant** → your **Subscription Name**.  
4. Under your subscription, navigate to **Storage Accounts** → **az104seaccount** → **Blob Containers** and **File Shares**.  

> _Screenshot: Show the storage account expanded under “Subscriptions.”_

---

## Task 8: Verify You Can Browse and Manage All Storage Resources

1. Under **Subscriptions** → your **Subscription** → **Storage Accounts** → **az104seaccount**:  
   - Expand **Blob Containers** and confirm you can see all blob containers.  
   - Expand **File Shares** and confirm you can see all file shares.  
2. Repeat **Tasks 5 and 6** using this full-account connection instead of a connection string or SAS:
   - Upload a different blob to any container.  
   - Upload a different file to any file share.  
   - Delete those items.  

> _Screenshot: Show you successfully uploaded and deleted a blob and a file using the subscription-connected account._

---

## Deliverables

1. **Storage Explorer Installation Screenshot**  
   - Show Azure Storage Explorer’s main window after installation.  

2. **Subscription Connection Screenshot**  
   - Show your subscription expanded under **Subscriptions** in Storage Explorer.  

3. **Connection String Screenshot**  
   - Show the storage account connected under **Storage Accounts (connected via Connection string)**.  

4. **Blob Upload/Delete Screenshots**  
   - Show `testfile.txt` uploaded to the blob container.  
   - Show the container list after deleting `testfile.txt`.  

5. **File Share Upload/Delete Screenshots**  
   - Show `testfile.txt` uploaded to the file share.  
   - Show the file share list after deleting `testfile.txt`.  

6. **SAS Connection Screenshot**  
   - Show the file share connected under **File Shares (connected via SAS)**.  

7. **Full Subscription Connection Screenshots**  
   - Show **az104seaccount** under **Subscriptions** → **Storage Accounts**.  
   - Show blob and file operations performed via the subscription connection.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may remove any test blobs/files you created. There is no need to uninstall Storage Explorer.

1. In Storage Explorer, delete any test blobs from each container:  
   - Right-click → **Delete**.  
2. Delete any test files from each file share:  
   - Right-click → **Delete**.  

> The storage account and resource group can remain; there are no additional resources created.

---

## Tips

- When connecting via **Connection string**, be sure to copy the entire string (including “DefaultEndpointsProtocol=…”).  
- When generating a **SAS URL**, grant only the permissions you need (Read/Write/Delete) and set a short expiry.  
- Use **Refresh** (F5) in Storage Explorer if you don’t see newly created or deleted items immediately.  
- The **Subscriptions** view requires Azure AD authentication; if you’re using a different account for subscription access, sign out and re-authenticate.  
- If you encounter a timeout or “permission denied,” verify you used the correct connection string or SAS, and that your Azure AD account has Storage Blob Data Contributor or similar role.

Good luck completing the lab tasks!  
