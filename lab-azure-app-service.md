
# AZ-104 Lab Instructions: Deploy a Simple App to Azure App Service (Azure Portal)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Create an App Service Plan  
3. Create a Web App with a built-in runtime (Node.js)  
4. Deploy a simple “Hello World” Node.js application from GitHub  
5. Verify the application is running in the browser  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups, App Service Plans, and Web Apps.  
- A user account that can log into the Azure Portal.  
- A GitHub account (for accessing the sample repository).  

---

## Lab Variables (Use Exactly These Values)

| Variable                     | Value                                                  |
|------------------------------|--------------------------------------------------------|
| **Region**                   | East US                                                |
| **Resource Group Name**      | az104-appsvc-rg                                        |
| **App Service Plan Name**    | az104-app-plan                                         |
| **App Service Plan SKU/Size**| S1 (Standard tier, 1 core, 1.75 GB RAM)                |
| **Web App Name**             | az104-nodejs-webapp                                    |
| **Runtime Stack**            | Node 14 LTS                                            |
| **GitHub Repo URL**          | https://github.com/Azure-Samples/nodejs-docs-hello-world |
| **GitHub Branch**            | master                                                 |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-appsvc-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Create App Service Plan

1. In the Azure Portal, click the search bar at the top and type **App Service plans**, then select **App Service plans**.  
2. Click **+ Create**.  
3. Under **Basics**, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-appsvc-rg**  
   - **Name**: `az104-app-plan`  
   - **Region**: `East US`  
   - **Operating System**: **Linux**  
   - **Pricing tier**: Click **Change size**, select **S1 (Standard)**, then click **Apply**  
4. Click **Review + create**, verify all settings match the variables above, then click **Create**.  
5. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Create Web App (Node.js Runtime)

1. In the Azure Portal, click the search bar and type **App Services**, then select **App Services**.  
2. Click **+ Create** and choose **Azure Web App**.  
3. Under the **Basics** tab, configure:  
   - **Subscription**: Your subscription  
   - **Resource group**: Select **az104-appsvc-rg**  
   - **Name**: `az104-nodejs-webapp`  
   - **Publish**: **Code**  
   - **Runtime stack**: **Node 14 LTS**  
   - **Region**: **East US**  
   - **Operating System**: **Linux**  
   - **App Service Plan**: Click **Change**, select **az104-app-plan**, then click **Select**  
4. Click **Review + create**, verify all settings match the variables above, then click **Create**.  
5. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 4: Deploy Sample “Hello World” Node.js App from GitHub

1. In the Azure Portal, navigate to **Resource groups** and select **az104-appsvc-rg**.  
2. Under **Resources**, click **az104-nodejs-webapp** to open the Web App blade.  
3. In the left-hand menu of the Web App blade, scroll down to **Deployment** and click **Deployment Center**.  
4. Under **Source**, select **GitHub**.  
5. Click **Authorize** (if prompted) to grant Azure permission to access your GitHub account.  
6. After authorization, under **Organization**, choose your GitHub user or organization.  
7. Under **Repository**, select **Azure-Samples/nodejs-docs-hello-world**.  
8. Under **Branch**, select **master**.  
9. In **Build Provider**, ensure **GitHub Actions** is selected (default).  
10. Click **Save** to configure continuous deployment.  
11. Azure will create (or update) a GitHub Actions workflow in the sample repo to deploy to your Web App.  
12. Wait until the Deployment Center shows **“Deployment succeeded”** under the **Logs** tab.  

> **Note:** It can take several minutes for GitHub Actions to complete the build and deploy. Refresh the Deployment Center logs until you see **“Deployment succeeded”**.

---

## Task 5: Verify the Running Web App

1. In the Azure Portal, go to **App Services** and select **az104-nodejs-webapp**.  
2. On the **Overview** page, copy the **Default domain** (e.g., `https://az104-nodejs-webapp.azurewebsites.net`).  
3. Open your web browser and navigate to the copied URL.  
4. Confirm you see the “Hello World” message from the sample Node.js application:  
```

Hello, World!

```
5. Take a screenshot of the browser window displaying the “Hello World” page.

---

## Deliverables

1. **Resource Group Screenshot**  
- Show **`az104-appsvc-rg`** in **East US** with provisioning state **“Succeeded.”**  

2. **App Service Plan Screenshot**  
- Show **`az104-app-plan`** in **East US** with **SKU = S1** under the App Service Plans blade.  

3. **Web App Overview Screenshot**  
- Show **`az104-nodejs-webapp`** in **East US** with **Runtime = Node 14 LTS** and **App Service Plan = az104-app-plan** on the Web App’s **Overview** page.  

4. **Deployment Center Logs Screenshot**  
- Show **“Deployment succeeded”** in the **Deployment Center** logs for **`az104-nodejs-webapp`**.  

5. **Browser Screenshot**  
- Show the “Hello, World!” page at `https://az104-nodejs-webapp.azurewebsites.net`.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-appsvc-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-appsvc-rg`** to confirm, then click **Delete**.  

> This action permanently deletes the App Service Plan, Web App, and any other resources under **`az104-appsvc-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, runtime stack, and plan—any deviation will affect grading.  
- When creating the Web App, ensure you explicitly select **Node 14 LTS** and **Linux** as the runtime.  
- If the Deployment Center fails, check the **Logs** tab for errors in the GitHub Actions workflow.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.  

Good luck completing the lab tasks!  
