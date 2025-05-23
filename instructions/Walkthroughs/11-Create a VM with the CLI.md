---
wts:
    title: 'Create a VM with the CLI'
    module: 'Module 02 - Core Azure Services'
---
# Create a VM with the CLI

In this walk-through, we will configure the Cloud Shell, use Azure CLI to create a resource group and virtual machine, and review Azure Advisor recommendations. 

# Task 1: Configure the Cloud Shell

In this task, we will configure Cloud Shell. 

1. Sign in to the [Azure portal](https://portal.azure.com).

2. From the Azure portal, open the **Azure Cloud Shell** by clicking on the icon in the top right of the Azure Portal.

    ![Screenshot of Azure Portal Azure Cloud Shell icon.](../images/1002.png)

3. If you have previously used the Cloud Shell, proceed to the next task. 

4. When prompted to select either **Bash** or **PowerShell**, select **Bash**. 

5. When prompted, click **Create storage**, and wait for the Azure Cloud Shell to initialize. 

# Task 2: Create a resource group and a virtual machine

In this task, we will use Azure CLI to create a resource group and a virtual machine.  

1. Ensure **Bash** is selected in the upper-left drop-down menu of the Cloud Shell pane (and if not, select it).

    ![Screenshot of Azure Portal Azure Cloud Shell with the Bash dropdown highlighted.](../images/1002a.png)

2. In the Bash session, within the Cloud Shell pane, create a new resource group. 

    ```cli
    resourcegroup="myResourceGroupVMCLI"
    location="westeurope"
    az group create --name $resourcegroup --location $location
    ```

3. Verify the resource group was created.

    ```cli
    az group list --output table
    ```

4. Create a new virtual machine. Make sure that each line except for the last one is followed by the backslash (`\`) character. If you type the whole command on the same line, do not use any backslash characters. 

    ```cli
    vmname="myVM"
    username="azureuser"
    az vm create \
    --resource-group $resourcegroup \
    --name $vmname \
    --image Ubuntu2204 \
    --size Standard_DS1_v2 \
    --public-ip-sku Standard \
    --admin-username $username \
    --authentication-type all \
    --generate-ssh-keys
    ```

    >**Note**: If you are using the command line on a Windows computer, replace the backslash (`\`) character with the caret (`^`) character.
    
    >**Note**: Your are requested for an admin password. Password requirements when creating a VM: between 12 – 123 characters; have lower characters; have upper characters; have a digit; have a special character.
    
    >**Note**: When the --size parameter is not specified when creating a virtual machine in Azure using Azure CLI, the default virtual machine size taken is Standard_DS1_v2. This size includes 1 vCPU and 3.5 GiB of memory, which is adequate for many development and test applications. If you get an error that says "Standard_DS1_v2 is currently not available in location westeurope", first registry the resource provider
    ```cli
	az provider register --namespace Microsoft.Compute
    ```
    >**Note**:If this does not work either, check if this size if available in your location
    ```cli
	az vm list-sizes --location $location --output table | grep Standard_DS1_v2
    ```
    >**Note**:If it is not available, try other size (e.g. Standard_B1s):
    ```cli
    vmname="myVM"
    username="azureuser"
    az vm create \
    --resource-group $resourcegroup \
    --name $vmname \
    --image Ubuntu2204 \
    --size Standard_B1s \
    --public-ip-sku Standard \
    --admin-username $username \
    --authentication-type all \
    --generate-ssh-keys    
    ```
    
    >**Note**: The command will take 2 to 3 minutes to complete. The command will create a virtual machine and various resources associated with it such as storage, networking and security resources. Do not continue to the next step until the virtual machine deployment is complete. 


5. Execute the command az vm extension set to confiture Nginx in your VM:
    ```cli
	az vm extension set \
	--resource-group $resourcegroup \
	--vm-name $vmname \
	--name customScript \
	--publisher Microsoft.Azure.Extensions \
	--version 2.1 \
	--settings "{\"fileUris\":[\"https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh\"]}" \
	--protected-settings "{\"commandToExecute\": \"./configure-nginx.sh\"}"
    ```

    >**Note**:he content of the file is as follows:
    
	```cli
	#!/bin/bash
	
	# Update apt cache.
	sudo apt-get update
	
	# Install Nginx.
	sudo apt-get install -y nginx
	
	# Set the home page.
	echo "<html><body><h2>Welcome to Azure! My name is $(hostname).</h2></body></html>" | sudo tee -a /var/www/html/index.html
	```

6. Open port 80 in the network settings
   ```cli
   az vm open-port --resource-group $resourcegroup --name $vmname --port 80
   ```

# Task 3: Test your application

1. In the Azure portal, search for **Virtual machines** and verify that your VM is running. Copy the Public IP address. Alternatively, you can use this command:

    ```cli
	az vm show --resource-group $resourcegroup --name $vmname -d --query [publicIps] --output tsv
    ```

2. Open a browser

# Task 4: Check that you can connect to the VM using SSH
 
1. SSH connect
    ```cli
    ssh azureuser@**your_ip**
    ```
2. Now you are in the bash of your virtual machine. If you want to exit, write exit.

# Task 5: Execute commmands in the Cloud Shell

In this task, we will practice executing CLI commands from the Cloud Shell. 

1. From the Azure portal, open the **Azure Cloud Shell** by clicking on the icon in the top right of the Azure Portal.

2. Ensure **Bash** is selected in the upper-left drop-down menu of the Cloud Shell pane.

3. Retrieve information about the virtual machine you provisioned, including name, resource group, location, and status. Notice the PowerState is **running**.

    ```cli
    az vm show --resource-group $resourcegroup --name $vmname --show-details --output table 
    ```

4. Stop the virtual machine. Notice the message that billing continues until the virtual machine is deallocated. 

    ```cli
    az vm stop --resource-group $resourcegroup --name $vmname
    ```

5. Verify your virtual machine status. The PowerState should now be **stopped**.

    ```cli
    az vm show --resource-group $resourcegroup --name $vmname --show-details --output table 
    ```

# Task 6: Review Azure Advisor Recommendations

In this task, we will review Azure Advisor recommendations.

   **Note:** If you have completed the previous lab (Create a VM with PowerShell), then you have already performed this task. 

1. From the **All services** blade, search for and select **Advisor**. 

2. On the **Advisor** blade, select **Overview**. Notice recommendations are grouped by High Availability, Security, Performance, and Cost. 

    ![Screenshot of the Advisor Overview page. ](../images/1103.png)

3. Select **All recommendations** and take time to view each recommendation and suggested actions. 

    **Note:** Depending on your resources, your recommendations will be different. 

    ![Screenshot of the Advisor All recommendations page. ](../images/1104.png)

4. Notice that you can download the recommendations as a CSV or PDF file. 

5. Notice that you can create alerts. 

6. If you have time, continue to experiment with Azure CLI. 

Congratulations! You have configured Cloud Shell, created a virtual machine using Azure CLI, practiced with Azure CLI commands, and viewed Advisor recommendations.

**Note**: To avoid additional costs, you can remove this resource group. Search for resource groups, click your resource group, and then click **Delete resource group**. Verify the name of the resource group and then click **Delete**. Monitor the **Notifications** to see how the delete is proceeding.
