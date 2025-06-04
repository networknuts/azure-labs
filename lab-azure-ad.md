
# AZ-104 Lab Instructions: Azure Entra ID (AD) – User Creation, Permissions & Group Management, Bulk Upload

In this lab, you will use the Azure Portal to complete the following tasks in Azure Entra ID (formerly Azure AD):

1. Create individual users  
2. Assign directory roles (permissions) to users  
3. Create security groups and add users to groups  
4. Perform a bulk upload of multiple users from a CSV file  

> **Note:** All steps must be performed through the Azure Portal GUI—do not use PowerShell or CLI.

---

## Prerequisites

- An Azure subscription with an Azure AD tenant  
- An account with **Global Administrator** or **User Administrator** privileges in Azure AD  
- A Windows or macOS machine with a browser  
- A local CSV file (`bulk-users.csv`) containing user details for bulk upload  

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value / Format                                                |
|-------------------------------|---------------------------------------------------------------|
| **Domain**                    | `<yourtenant>.onmicrosoft.com`                                |
| **User1 Display Name**        | `John Doe`                                                    |
| **User1 Username (UPN)**      | `johndoe@<yourtenant>.onmicrosoft.com`                        |
| **User1 Initial Password**    | `P@ssw0rd1234!`                                               |
| **User2 Display Name**        | `Jane Smith`                                                  |
| **User2 Username (UPN)**      | `janesmith@<yourtenant>.onmicrosoft.com`                      |
| **User2 Initial Password**    | `P@ssw0rd1234!`                                               |
| **Role to Assign**            | `User Administrator`                                          |
| **Security Group Name**       | `Demo-Security-Group`                                         |
| **Bulk CSV File Name**        | `bulk-users.csv`                                              |
| **CSV Columns (header)**      | `DisplayName,UserPrincipalName,MailNickname,Password`         |

_Do not change these values or formats; the lab validation depends on them._

---

## Task 1: Create Individual Azure AD Users

### 1.1 Create User “John Doe”

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Azure Active Directory**.  
3. Under **Manage**, click **Users** → **+ New user** → **Create user**.  
4. In the **Create user** pane, configure:  
   - **User name**: `johndoe@<yourtenant>.onmicrosoft.com`  
   - **Name**: `John Doe`  
   - **Password**: Select **Let me create the password** and enter `P@ssw0rd1234!` (uncheck **Show password** after confirming).  
   - Leave **Groups and roles** blank for now.  
   - Under **Properties**, leave **Block sign in** = **No**.  
5. Click **Create**.  
6. Verify that **John Doe** appears in the user list.  

   > _Screenshot:_ Show **John Doe** in the **Users** list with **User Principal Name** `johndoe@<yourtenant>.onmicrosoft.com`.

### 1.2 Create User “Jane Smith”

1. Repeat steps 3–6 above, replacing values:  
   - **User name**: `janesmith@<yourtenant>.onmicrosoft.com`  
   - **Name**: `Jane Smith`  
   - **Password**: `P@ssw0rd1234!`  
2. Verify **Jane Smith** appears in the user list.  

   > _Screenshot:_ Show **Jane Smith** in the **Users** list with **User Principal Name** `janesmith@<yourtenant>.onmicrosoft.com`.

---

## Task 2: Assign a Directory Role to a User

You will assign the **User Administrator** role to **John Doe** so he can manage users (but not global settings).

1. In **Azure Active Directory**, click **Roles and administrators** under **Manage**.  
2. In the **Roles and administrators** list, select **User administrator**.  
3. Click **+ Add assignments**.  
4. In the **Add assignment** pane, click **None selected** → search for **John Doe** → select **John Doe** → click **Select** → **Add**.  
5. Confirm that **John Doe** appears under **Assigned** for the **User administrator** role.  

   > _Screenshot:_ Show **User administrator** role blade with **John Doe** under **Assigned**.

---

## Task 3: Create a Security Group and Add Users

### 3.1 Create a Security Group “Demo-Security-Group”

1. In **Azure Active Directory**, click **Groups** under **Manage** → **+ New group**.  
2. In the **Create a group** pane, configure:  
   - **Group type**: **Security**  
   - **Group name**: `Demo-Security-Group`  
   - **Group description**: `Group for demo users.`  
   - **Membership type**: **Assigned**  
3. Click **Create**.  
4. Verify that **Demo-Security-Group** appears in the group list.  

   > _Screenshot:_ Show **Demo-Security-Group** in the **Groups** list with **Membership type = Assigned**.

### 3.2 Add Users to “Demo-Security-Group”

1. In **Groups**, click **Demo-Security-Group** → **Members** → **+ Add members**.  
2. In the **Add members** pane, search for and select **John Doe** and **Jane Smith** → click **Select**.  
3. Verify that both **John Doe** and **Jane Smith** appear under **Members**.  

   > _Screenshot:_ Show **Demo-Security-Group** > **Members** listing **John Doe** and **Jane Smith**.

---

## Task 4: Bulk Upload Users via CSV

Azure AD allows bulk creation of up to 20,000 users via CSV import.

### 4.1 Prepare the CSV File

1. On your local workstation, create a file named `bulk-users.csv` with the following header row and user details (replace `<yourtenant>` accordingly):

   ```csv
   DisplayName,UserPrincipalName,MailNickname,Password
   Alice Adams,aliceadams@<yourtenant>.onmicrosoft.com,aliceadams,P@ssw0rd1234!
   Bob Baker,bobbaker@<yourtenant>.onmicrosoft.com,bobbaker,P@ssw0rd1234!
   Carol Clark,carolclark@<yourtenant>.onmicrosoft.com,carolclark,P@ssw0rd1234!
   ```

2. Save `bulk-users.csv` in a known local folder.
3. Verify the CSV is properly formatted (UTF-8, no extra columns).

   > *Screenshot:* Show the contents of `bulk-users.csv` in a text editor.

### 4.2 Upload CSV to Bulk Create Users

1. In **Azure Active Directory**, click **Users** → **Bulk create** (top menu).
2. In the **Bulk create** blade:

   * Click **Download a template** → choose **Download**.
   * Ensure the template matches your `bulk-users.csv` header and format.
   * Click **Browse**, select your local `bulk-users.csv`.
   * Click **Submit**.
3. Wait for Azure to process the CSV. A notification will appear indicating success or errors.
4. Under **Bulk operations** → **Bulk user activity**, click **History** → select the most recent run → verify **Status = Completed** with 3 users created.

   > *Screenshot:* Show **Bulk user activity** → **History** with Status = Completed, Users created: 3.

### 4.3 Verify Bulk-Uploaded Users

1. In **Azure Active Directory** → **Users**, search for each of:

   * `aliceadams@<yourtenant>.onmicrosoft.com`
   * `bobbaker@<yourtenant>.onmicrosoft.com`
   * `carolclark@<yourtenant>.onmicrosoft.com`
2. Confirm all three appear in the user list with **User Principal Name** matching.

   > *Screenshot:* Show all three bulk-uploaded users in the **Users** list.

---

## Deliverables

1. **User Creation Screenshots**

   * **John Doe** and **Jane Smith** in the **Users** list with correct UPNs.
   * **User administrator** role blade showing **John Doe** under **Assigned**.

2. **Group Management Screenshots**

   * **Demo-Security-Group** in the **Groups** list with **Membership type = Assigned**.
   * **Demo-Security-Group** > **Members** listing **John Doe** and **Jane Smith**.

3. **Bulk Upload Preparation Screenshot**

   * Text editor showing `bulk-users.csv` contents with header and three user rows.

4. **Bulk Create Operation Screenshots**

   * **Bulk create** blade showing the CSV upload in progress or completed.
   * **Bulk user activity** → **History** showing **Status = Completed**, 3 users processed.

5. **Bulk-Uploaded Users Verification**

   * **Users** list showing **Alice Adams**, **Bob Baker**, and **Carol Clark** with correct UPNs.

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete test users and groups:

1. In **Azure Active Directory** → **Users**:

   * Delete **John Doe**, **Jane Smith**, **Alice Adams**, **Bob Baker**, and **Carol Clark** (select each → **Delete user**).

2. In **Azure Active Directory** → **Groups**:

   * Delete **Demo-Security-Group** (select → **Delete**).

> Deleting users/groups ensures no lingering test accounts.

---

## Lab Tips

* User Principal Names (UPNs) must be unique in the tenant and follow email address format.
* Passwords in the CSV must meet Azure AD complexity requirements (minimum 8 characters, mix of letters, numbers, symbols).
* When assigning roles, a user must sign out and back in for role privileges to take effect.
* Bulk upload errors (e.g., duplicate UPN or invalid formatting) appear in the **Bulk user activity** details; correct the CSV and retry.
* For larger imports (>5,000 users), allow extra time for processing.
* Always verify newly created users in the **Users** list before proceeding with group assignments or role changes.

Good luck completing the lab tasks!

