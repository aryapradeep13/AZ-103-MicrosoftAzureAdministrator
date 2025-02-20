---
lab:
    title: 'Deploy and Manage Virtual Machines'
    module: 'Module 02 - Azure Virtual Machines'
---

# Lab: Deploy and Manage Virtual Machines

All tasks in this lab are performed from the Azure portal (including a PowerShell Cloud Shell session) except for Exercise 2 Task 2 and Exercise 2 Task 3, which include steps performed from a Remote Desktop session to an Azure VM

   > **Note**: When not using Cloud Shell, the lab virtual machine must have Azure PowerShell module installed [**https://docs.microsoft.com/en-us/powershell/azure/install-Az-ps**](https://docs.microsoft.com/en-us/powershell/azure/install-Az-ps)

Lab files: 

-  **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_azuredeploy.json**

-  **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_azuredeploy.parameters.json**

-  **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_install_iis_vmss.zip**

### Scenario
  
Adatum Corporation wants to implement its workloads by using Azure virtual machines (VMs) and Azure VM scale sets


### Objectives
  
After completing this lab, you will be able to:

-  Deploy Azure VMs by using the Azure portal, Azure PowerShell, and Azure Resource Manager templates

-  Configure networking settings of Azure VMs running Windows and Linux operating systems

-  Deploy and configure Azure VM scale sets


### Exercise 1: Deploy Azure VMs by using the Azure portal, Azure PowerShell, and Azure Resource Manager templates
  
The main tasks for this exercise are as follows:

1. Deploy an Azure VM running Windows Server 2016 Datacenter into an availability set by using the Azure portal

1. Deploy an Azure VM running Windows Server 2016 Datacenter into the existing availability set by using Azure PowerShell

1. Deploy two Azure VMs running Linux into an availability set by using an Azure Resource Manager template


#### Task 1: Deploy an Azure VM running Windows Server 2016 Datacenter into an availability set by using the Azure portal

1. From the lab virtual machine, start Microsoft Edge, browse to the Azure portal at [**http://portal.azure.com**](http://portal.azure.com) and sign in by using a Microsoft account that has the Owner role in the Azure subscription you intend to use in this lab.

1. In the Azure portal, navigate to the **Create a resource** blade.

1. From the **Create a resource** blade, search Azure Marketplace for **Windows Server**. Select **Windows Server** from the search results list.

1. On the Windows Server page, use the drop-down menu to select **[smalldisk] Windows Server 2016 Datacenter**, and then click **Create**.

1. Use the **Create a virtual machine** blade to deploy a virtual machine with the following settings:

    - Subscription: the name of the subscription you are using in this lab

    - Resource group: the name of a new resource group **az1000301-RG**

    - Virtual machine name: **az1000301-vm0**

    - Region: **(US) East US** (or a region closer to you)

      > **Note**: To identify Azure regions where you can provision Azure VMs, refer to [**https://azure.microsoft.com/en-us/regions/offers/**](https://azure.microsoft.com/en-us/regions/offers/)

    - Availability options: **Availability set**

    - Availability set: Click **Create New**, and name the new availability set **az1000301-avset0** with **2** fault domains and **5** update domains. Click **OK**.

    - Image: **[smalldisk] Windows Server 2016 Datacenter**

    - Size: **Standard DS2_v2**

    - Username: **Student**

    - Password: **Pa55w.rd1234**

    - Public inbound ports: **None**

    - Already have a Windows license?: **No**

1. Click **Next: Disks >**.    

    - OS disk type: **Standard HDD**

1. Click **Next: Networking >**.

1. On the Networking tab, click **Create new** under Virtual Network. Use the virtual network name already assigned by default and specify the following:

    - Virtual network address range: **10.103.0.0/16**

    - Subnet name: **subnet0**

    - Subnet address range: **10.103.0.0/24**

1. Click **OK**.

1. Click **Next: Management >**.

1. On the Management tab, review the default settings and note that boot diagnostics are turned on with a new diagnostics storage account automatically preconfigured.

1. Click **Next: Advanced >**.

1. On the Advanced tab, review the available settings.

1. Leave all settings with their default values, and click **Review + create**.

1. Click **Create**.

   > **Note**: You will configure the network security group you create in this task in the second exercise of this lab

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This should take about 5 minutes.


#### Task 2: Deploy an Azure VM running Windows Server 2016 Datacenter into the existing availability set by using Azure PowerShell

1. From the Azure Portal, start a PowerShell session in the Cloud Shell pane. 

   > **Note**: If this is the first time you are launching the Cloud Shell in the current Azure subscription, you will be asked to create an Azure file share to persist Cloud Shell files. If so, accept the defaults, which will result in creation of a storage account in an automatically generated resource group.

1. In the Cloud Shell pane, run the following command:

   ```pwsh
   $vmName = 'az1000301-vm1'
   $vmSize = 'Standard_DS2_v2'
   ```

   > **Note**: This sets the values of variables designating the Azure VM name and its size

1. In the Cloud Shell pane, run the following commands:

   ```pwsh
   $resourceGroup = Get-AzResourceGroup -Name 'az1000301-RG'
   $location = $resourceGroup.Location
   ```

   > **Note**: These commands set the values of variables designating the target resource group and its location

1. In the Cloud Shell pane, run the following commands:

   ```pswh
   $availabilitySet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup.ResourceGroupName -Name 'az1000301-avset0'
   $vnet = Get-AzVirtualNetwork -Name 'az1000301-RG-vnet' -ResourceGroupName $resourceGroup.ResourceGroupName
   $subnetid = (Get-AzVirtualNetworkSubnetConfig -Name 'subnet0' -VirtualNetwork $vnet).Id
   ```

   > **Note**: These commands set the values of variables designating the availability set, virtual network, and subnet into which you will deploy the new Azure VM

1. In the Cloud Shell pane, run the following commands:

   ```pwsh
   $nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -Name "$vmName-nsg"
   $pip = New-AzPublicIpAddress -Name "$vmName-ip" -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -AllocationMethod Dynamic 
   $nic = New-AzNetworkInterface -Name "$($vmName)$(Get-Random)" -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -SubnetId $subnetid -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
   ```

   > **Note**: These commands create a new network security group, public IP address, and network interface that will be used by the new Azure VM

   > **Note**: You will configure the network security group you create in this task in the second exercise of this lab

1. In the Cloud Shell pane, run the following commands:

   ```pwsh
   $adminUsername = 'Student'
   $adminPassword = 'Pa55w.rd1234'
   $adminCreds = New-Object PSCredential $adminUsername, ($adminPassword | ConvertTo-SecureString -AsPlainText -Force)
   ```

   > **Note**: These commands set the values of variables designating credentials of the local Administrator account of the new Azure VM

1. In the Cloud Shell pane, run the following commands:

   ```pwsh
   $publisherName = 'MicrosoftWindowsServer'
   $offerName = 'WindowsServer'
   $skuName = '2016-Datacenter'
   ```

   > **Note**: These commands set the values of variables designating the properties of the Azure Marketplace image that will be used to provision the new Azure VM

1. In the Cloud Shell pane, run the following command:

   ```pwsh
   $osDiskType = (Get-AzDisk -ResourceGroupName $resourceGroup.ResourceGroupName)[0].Sku.Name
   ```

   > **Note**: This command sets the values of a variable designating the operating system disk type of the new Azure VM

1. In the Cloud Shell pane, run the following commands:

   ```pwsh
   $vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $availabilitySet.Id
   Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
   Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential $adminCreds 
   Set-AzVMSourceImage -VM $vmConfig -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version 'latest'
   Set-AzVMOSDisk -VM $vmConfig -Name "$($vmName)_OsDisk_1_$(Get-Random)" -StorageAccountType $osDiskType -CreateOption fromImage
   Set-AzVMBootDiagnostic -VM $vmConfig -Disable
   ```

   > **Note**: These commands set up the properties of the Azure VM configuration object that will be used to provision the new Azure VM, including the VM size, its availability set, network interface, computer name, local Administrator credentials, the source image, the operating system disk, and boot diagnostics settings.

1. In the Cloud Shell pane, run the following command:

   ```pwsh
   New-AzVM -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -VM $vmConfig
   ```

   > **Note**: This command initiates deployment of the new Azure VM

   > **Note**: Do not wait for the deployment to complete but instead proceed to the next task.


#### Task 3: Deploy two Azure VMs running Linux into an availability set by using an Azure Resource Manager template

1. In the Azure portal, navigate to the **Create a resource** blade.

1. From the **Create a resource** blade, search Azure Marketplace for **Template deployment**, and select **Template deployment (deploy using custom templates)**.

1. Click **Create**.

1. On the **Custom deployment** blade, click the **Build your own template in the editor** link. If you do not see this link, click **Edit template** instead.

1. From the **Edit template** blade, load the template file **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_azuredeploy.json**. 

   > **Note**: Review the content of the template and note that it defines deployment of two Azure VMs hosting Linux Ubuntu into an availability set and into the existing virtual network **az1000301-vnet0**. This virtual network does not exist in your deployment. You will be changing the virtual network name in the parameters below.

1. Save the template and return to the **Custom deployment** blade. 

1. From the **Custom deployment** blade, navigate to the **Edit parameters** blade.

1. From the **Edit parameters** blade, load the parameters file **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_azuredeploy.parameters.json**. 

1. Save the parameters and return to the **Custom deployment** blade. 

1. From the **Custom deployment** blade, initiate a template deployment with the following settings:

    - Subscription: the name of the subscription you are using in this lab

    - Resource group: the name of a new resource group **az1000302-RG**

    - Location: the same Azure region you chose earlier in this exercise

    - Vm Name Prefix: **az1000302-vm**

    - Nic Name Prefix: **az1000302-nic**

    - Pip Name Prefix: **az1000302-ip**

    - Admin Username: **Student**

    - Admin Password: **Pa55w.rd1234**

    - Virtual Network Name: **az1000301-RG-vnet** _(change this value from the template default)_

    - Image Publisher: **Canonical**

    - Image Offer: **UbuntuServer**

    - Image SKU: **16.04.0-LTS**

    - Vm Size: use **Standard_DS1_v2** or **Standard_DS2_v2**, based on the instructor's recommendations 

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This should take about 5 minutes.

> **Result**: After you completed this exercise, you have deployed an Azure VM running Windows Server 2016 Datacenter into an availability set by using the Azure portal, deployed another Azure VM running Windows Server 2016 Datacenter into the same availability set by using Azure PowerShell, and deployed two Azure VMs running Linux Ubuntu into an availability set by using an Azure Resource Manager template.

   > **Note**: You could certainly use a template to deploy two Azure VMs hosting Windows Server 2016 datacenter in a single task (just as this was done with two Azure VMs hosting Linux Ubuntu server). The reason for deploying these Azure VMs in two separate tasks was to give you the opportunity to become familiar with both the Azure portal and Azure PowerShell-based deployments.


### Exercise 2: Configure networking settings of Azure VMs running Windows and Linux operating systems
  
The main tasks for this exercise are as follows:

1. Configure static private and public IP addresses of Azure VMs

1. Connect to an Azure VM running Windows Server 2016 Datacenter via a public IP address

1. Connect to an Azure VM running Linux Ubuntu Server via a private IP address


#### Task 1: Configure static private and public IP addresses of Azure VMs

1. In the Azure portal, navigate to the **az1000301-vm0** blade.

1. From the **az1000301-vm0** blade, navigate to the **Networking** blade, displaying the configuration of the public IP address **az1000301-vm0-ip**, assigned to its network interface.

1. From the **Networking** blade, click the link representing the public IP address.

1. On the az1000301-vm0-ip blade, click **Configuration**.

1. Change the assignment of the public IP address to **Static**, and then click **Save**.

   > **Note**: Take a note of the public IP address assigned to the network interface of **az1000301-vm0**. You will need it later in this exercise.

1. In the Azure portal, navigate to the **az1000302-vm0** blade.

1. From the **az1000302-vm0** blade, display the **Networking** blade.

1. On the **az1000302-vm0 - Networking** blade, click the entry representing network interface (with name starting with az1000302-vm0).

1. From the blade displaying the properties of the network interface of **az1000302-vm0**, navigate to its **IP configurations** blade.

1. On the **IP configurations** blade, configure the **ipconfig1** private IP address to be static and set it to **10.103.0.100**, and then click **Save**.

   > **Note**: Changing the private IP address assignment requires restarting the Azure VM.

   > **Note**: It is possible to connect to Azure VMs via either statically or dynamically assigned public and private IP addresses. Choosing static IP assignment is commonly done in scenarios where these IP addresses are used in combination with IP filtering, routing, or if they are assigned to network interfaces of Azure VMs that function as DNS servers.


#### Task 2: Connect to an Azure VM running Windows Server 2016 Datacenter via a public IP address

1. In the Azure portal, navigate to the **az1000301-vm0** blade.

1. From the **az1000301-vm0** blade, navigate to the **Networking** blade.

1. On the **az1000301-vm0 - Networking** blade, review the inbound port rules of the network security group assigned to the network interface of **az1000301-vm0**.

   > **Note**: The default configuration consisting of built-in rules block inbound connections from the internet (including connections via the RDP port TCP 3389)

1. Add an inbound security rule to the existing network security group with the following settings:

    - Source: **Any**

    - Source port ranges: **\***

    - Destination: **Any**

    - Destination port ranges: **3389**

    - Protocol: **TCP**

    - Action: **Allow**

    - Priority: **100**

    - Name: **AllowInternetRDPInBound**

1. In the Azure portal, display the **Overview** pane of the **az1000301-vm0** blade. 

1. From the **Overview** pane of the **az1000301-vm0** blade, click **Connect** and generate an RDP file and use it to connect to **az1000301-vm0**.

1. When prompted, authenticate by specifying the following credentials:

    - User name: **Student**

    - Password: **Pa55w.rd1234**


#### Task 3: Connect to an Azure VM running Linux Ubuntu Server via a private IP address
 
1. Within the RDP session to **az1000301-vm0**, start **Command Prompt**.

1. From the Command Prompt, run the following:

   ```
   nslookup az1000302-vm0
   ```

1. Examine the output and note that the name resolves to the IP address you assigned in the first task of this exercise (**10.103.0.100**).

   > **Note**: This is expected. Azure provides built-in DNS name resolution within a virtual network. 

1. Within the RDP session to **az1000301-vm0**, from Server Manager, click **Local Server**, then disable **IE Enhanced Security Configuration**.

1. Within the RDP session to **az1000301-vm0**, start Internet Explorer and download **putty.exe** from [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 

1. Use **putty.exe** to verify that you can successfully connect to **az1000302-vm0** on its private IP address via the **SSH** protocol (TCP 22).

1. When prompted, authenticate by specifying the following values:

    - User name: **Student**

    - Password: **Pa55w.rd1234**

    > **Note**: Both the username and password are case sensitive.
 
1. Once you successfully authenticated, terminate the RDP session to **az1000301-vm0**.

1. On the lab virtual machine, in the Azure portal, navigate to the **az1000302-vm0** blade.

1. From the **az1000302-vm0** blade, navigate to the **Networking** blade.

1. On the **az1000302-vm0 - Networking** blade, review the inbound port rules of the network security group assigned to the network interface of **az1000301-vm0** to determine why your SSH connection via the private IP address was successsful.

   > **Note**: The default configuration consisting of built-in rules allows inbound connections within the Azure virtual network environment (including connections via the SSH port TCP 22).

> **Result**: After you completed this exercise, you have configured static private and public IP addresses of Azure VMs, connected to an Azure VM running Windows Server 2016 Datacenter via a public IP address, and connect to an Azure VM running Linux Ubuntu Server via a private IP address


### Exercise 3: Deploy and configure Azure VM scale sets

The main tasks for this exercise are as follows:

1. Identify an available DNS name for an Azure VM scale set deployment

1. Deploy an Azure VM scale set

1. Install IIS on a scale set VM by using DSC extensions


#### Task 1: Identify an available DNS name for an Azure VM scale set deployment

1. From the Azure Portal, start a PowerShell session in the Cloud Shell pane. 

1. In the Cloud Shell pane, run the following command, substituting the placeholder &lt;custom-label&gt; with any string which is likely to be unique.

   ```pwsh
   $rg = Get-AzResourceGroup -Name az1000301-RG
   Test-AzDnsAvailability -DomainNameLabel <custom-label> -Location $rg.Location
   ```

1. Verify that the command returned **True**. If not, rerun the same command with a different value of the &lt;custom-label&gt; until the command returns **True**. 

1. Note the value of the &lt;custom-label&gt; that resulted in the successful outcome. You will need it in the next task


#### Task 2: Deploy an Azure VM scale set

1. In the Azure portal, navigate to the **Create a resource** blade.

1. From the **Create a resource** blade, search Azure Marketplace for **Virtual machine scale set**.

1. Use the list of search results to navigate to the **Create virtual machine scale set** blade.

1. Use the **Create virtual machine scale set** blade to deploy a virtual machine scale set with the following settings:

    - Virtual machine scale set name: **az1000303vmss0**

    - Operating system disk image: **Windows Server 2016 Datacenter**

    - Subscription: the name of the subscription you are using in this lab

    - Resource group: the name of a new resource group **az1000303-RG**

    - Location: the same Azure region you chose in the previous exercises of this lab

    - Availability zone: **None**

    - Username: **Student**

    - Password: **Pa55w.rd1234**

    - Instance count: **1**

    - Instance size: **DS2 v2**

    - Deploy as low priority: **No**

    - Use managed disks: **Yes**

    - Autoscale: **Disabled**

    - Choose Load balancing options: **Load balancer**

    - Public IP address name: **az1000303vmss0-ip**

    - Domain name label: type in the value of the &lt;custom-label&gt; you identified in the previous task

    - Virtual network: the name of a new virtual network **az1000303-vnet0** with the following settings:

        - Address range: **10.203.0.0/16**

        - Subnet name: **subnet0**

        - Subnet address range: **10.203.0.0/24**

    - Public IP address per instance: **Off**
    
    - Accelerated networking: **Off**
    
    - NIC network security group: **Basic**
    
    - Select inbound ports: **HTTP**
    
    - Boot diagnostics: **Off**
    
    - System assigned managed identity: **Off**

   > **Note**: Wait for the deployment to complete before you proceed to the next task. This should take about 5 minutes.


#### Task 3: Install IIS on a scale set VM by using DSC extensions

1. In the Azure portal, navigate to the **az1000303vmss0** blade.

1. From the **az1000303vmss0** blade, display its Extension blade.

1. From the **az1000303vmss0 - Extension** blade, add the **PowerShell Desired State Configuration** extension with the following settings:

   > **Note**: The DSC configuration module is available for upload from **Labfiles\\Module_02\\Deploy_and_Manage_Virtual_Machines\\az-100-03_install_iis_vmss.zip**. The module contains the DSC configuration script that installs the Web Server (IIS) role.

    - Configuration Modules or Script: **"az-100-03_install_iis_vmss.zip"**

    - Module-qualified Name of Configuration: **az-100-03_install_iis_vmss.ps1\\IISInstall**

    - Configuration Arguments: leave blank

    - Configuration Data PSD1 File: leave blank

    - WMF Version: **latest**

    - Data Collection: **Disable**

    - Version: **2.76**

    - Auto Upgrade Minor Version: **Yes**

1. Navigate to the **az1000303vmss0 - Instances** blade and initiate the upgrade of the **az1000303vmss0_0** instance.

   > **Note**: The update will trigger application of the DSC configuration script. Wait for upgrade to complete. This should take about 5 minutes. You can monitor the progress from the **az1000303vmss0 - Instances** blade.

1. Once the upgrade completes, navigate to the **Overview** blade. 

1. On the **az1000303vmss0-ip** blade, note the public IP address assigned to **az1000303vmss0**.

1. Start Microsoft Edge and navigate to the public IP address you identified in the previous step.

1. Verify that the browser displays the default IIS home page. 

> **Result**: After you completed this exercise, you have identified an available DNS name for an Azure VM scale set deployment, deployed an Azure VM scale set, and installed IIS on a scale set VM by using the DSC extension.

## Exercise 4: Remove lab resources

#### Task 1: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. At the Cloud Shell interface, select **Bash**.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list all resource groups you created in this lab:

   ```sh
   az group list --query "[?starts_with(name,'az1000')].name" --output tsv
   ```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Task 2: Delete resource groups

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the resource groups you created in this lab

   ```sh
   az group list --query "[?starts_with(name,'az1000')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Close the **Cloud Shell** prompt at the bottom of the portal.

> **Result**: In this exercise, you removed the resources used in this lab.
