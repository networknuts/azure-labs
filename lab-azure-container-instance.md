
# AZ-104 Lab Instructions: Deploy and Connect to an Azure Container Instance (ACI)

In this lab, you will use the Azure Portal (GUI) to complete the following tasks:

1. Create a new Resource Group  
2. Deploy an Azure Container Instance running the NGINX image  
3. Verify the container is running and connect to the default NGINX page  

_Do not use Azure CLI or PowerShell—complete every step through the Azure Portal interface._

---

## Prerequisites

- An active Azure subscription with permissions to create Resource Groups and Container Instances  
- A user account that can log into the Azure Portal  

---

## Lab Variables (Use Exactly These Values)

| Variable                      | Value                                   |
|-------------------------------|-----------------------------------------|
| **Region**                    | East US                                 |
| **Resource Group Name**       | az104-aci-rg                            |
| **Container Group Name**      | nginx-aci                               |
| **Container Name**            | nginx-container                         |
| **Container Image**           | nginx:latest                            |
| **CPU Cores**                 | 1                                       |
| **Memory (GiB)**              | 1                                       |
| **Port**                      | 80                                      |
| **DNS Label (FQDN prefix)**   | nginx-aci-demo                          |

_Do not change any of these values. If you modify names, regions, or sizes, your lab will not match the grading rubric._

---

## Task 1: Create Resource Group

1. Sign in to the [Azure Portal](https://portal.azure.com).  
2. In the left-hand menu, select **Resource groups**.  
3. Click **+ Create**.  
4. Under **Basics**, fill in:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: `az104-aci-rg`  
   - **Region**: `East US`  
5. Leave **Tags** blank.  
6. Click **Review + create**, then **Create**.  
7. Wait until you see **“Deployment succeeded”** before proceeding.

---

## Task 2: Deploy Azure Container Instance

1. In the Azure Portal, click **Create a resource** (upper-left corner).  
2. Search for and select **Container instances**, then click **Create**.  
3. Under the **Basics** tab, configure:  
   - **Subscription**: Your Azure subscription  
   - **Resource group**: Select **az104-aci-rg**  
   - **Container name**: `nginx-container`  
   - **Region**: `East US`  
   - **Image source**: **Docker Hub**  
   - **Image**: `nginx:latest`  
   - **Operating System**: **Linux**  
   - **CPU cores**: `1`  
   - **Memory**: `1 GiB`  
4. Under **Container port** (in the **Networking** section), set:  
   - **Port**: `80`  
5. Under **DNS name label**, enter: `nginx-aci-demo`   
   - This creates a fully qualified domain name (FQDN) of `nginx-aci-demo.eastus.azurecontainer.io`.  
6. Ensure **Public (Internet)** is selected for **Connectivity**.  
7. Click **Review + create**, verify all settings match the variables above, then click **Create**.  
8. Wait for **“Deployment succeeded”** before proceeding.

---

## Task 3: Verify and Connect to the Container

1. In the Azure Portal, navigate to **Resource groups**, then select **az104-aci-rg**.  
2. Under **Resources**, click **nginx-aci** (the Container Group).  
3. On the **Overview** page, verify the **Provisioning state** is **Succeeded** and the **State** is **Running**.  
   - Take a screenshot of the **Overview** blade showing **“Running”**.  
4. Copy the **FQDN** listed under **IP address / FQDN** (e.g., `nginx-aci-demo.eastus.azurecontainer.io`).  
5. Open your web browser and navigate to:  

[http://nginx-aci-demo.eastus.azurecontainer.io](http://nginx-aci-demo.eastus.azurecontainer.io)

6. Confirm you see the default NGINX welcome page.  
- Take a screenshot of the browser window showing the NGINX default page.

---

## Deliverables

1. **Resource Group Screenshot**  
- Show **`az104-aci-rg`** in **East US** with provisioning state **“Succeeded.”**  

2. **Container Group Overview Screenshot**  
- Show **`nginx-aci`** Container Group in **East US** with **State = Running** and the assigned **FQDN**.  

3. **Browser Screenshot**  
- Show the NGINX default welcome page at `http://nginx-aci-demo.eastus.azurecontainer.io`.  

---

## Cleanup (Optional)

After capturing all required screenshots, you may delete the entire resource group to remove all resources:

1. In the Azure Portal, navigate to **Resource groups**.  
2. Select **az104-aci-rg**.  
3. Click **Delete resource group**.  
4. Type **`az104-aci-rg`** to confirm, then click **Delete**.  

> This action permanently deletes the Container Instance and any other resources under **`az104-aci-rg`**.

---

## Lab Tips

- Copy and paste the exact variable values for names, region, image, CPU, memory, and port—any deviation will affect grading.  
- When setting the DNS name label, ensure it is unique within the **eastus.azurecontainer.io** namespace.  
- The FQDN will be displayed on the **Overview** blade; use it exactly as shown to connect.  
- All steps must be performed through the Azure Portal GUI—do not switch to CLI or PowerShell.  

Good luck completing the lab tasks!  
```
