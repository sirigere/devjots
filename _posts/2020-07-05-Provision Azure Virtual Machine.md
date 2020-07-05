---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204, IaaS]
identifier: Az204-Compute-VM
title: Provision VM and Configure for remote access
---

``Azure Virtual Machine`` is a ``IaaS`` (Infrastructure as a Service) offering from Azure. Every Virtual Machine is composed of multiple virtual resources. To Create an Virtual Machine, we can create these virtual services independently. Once done we can combine them to make a Virtual Machine. In this post, I am going to discuss these independent virtual services and how to provision them. There are many ways we can provision these Azure Virtual Machine. We will familiarize ourself with how we can create these resources using ``PowerShell``, ``Azure CLI`` and ``ARM``. Below are the main components of a Azure Virtual Machine.

+ Networking
+ Storage
+ Image

## Networking
Below hierarchical list shows how different components are dependent on each other.

+ Virtual Network (VNet)
  + Subnet
    + Network Security Group (NSG)
       + Network Security Group Rule
    + Network Interface (NIC)
      + Network Security Group (NSG)
        + Network Security Group Rule
      + Public Ip Address

#### Below Powershell script creates networking resources

<pre>
    <code class="powershell">
      $resourceGroupName = "sample-vm-rg"
      $location = "eastus"

      Connect-AzAccount   #Connect to your Azure account

      New-AzResourceGroup -Name $resourceGroupName -Location $location   #Create Resource Group

      $vnet = New-AzVirtualNetwork -Name "sample-vm-rg-vnet" -Location eastus  -ResourceGroupName "sample-vm-rg" -AddressPrefix "10.0.0.0/16"    #Create VNet

      Add-AzVirtualNetworkSubnetConfig -Name "fontend-servers" -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/24"   #Add subnet configuration with the virtual network

      $vnet | Set-AzVirtualNetwork #Persist the new subnet created

      $vnet = Get-AzVirtualNetwork -Name "sample-vm-rg-vnet" #Re-initial the vnet variable as we need the id of the newy added subnet

      $subnet = Get-AzVirtualNetworkSubnetConfig -Name "fontend-servers" -VirtualNetwork $vnet #Get the definition of subnet as we need this information in next steps

      $pip = New-AzPublicIpAddress -Name "sample-vm-rg-pip" -ResourceGroupName $resourceGroupName -Location $location -AllocationMethod Dynamic  #Create a public ip address

      $nic = New-AzNetworkInterface -Name "sample-vm-rg-nic" -Location $location -ResourceGroupName $resourceGroupName 
      -Subnet $subnet -PublicIpAddress $pip

      $allowRdpRule = New-AzNetworkSecurityRuleConfig -Name "Allow RDP" -Protocol Tcp -Direction Inbound -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389

      $nsg = New-AzNetworkSecurityGroup -Name "sample-vm-rg-nsg" -ResourceGroupName $resourceGroupName -Location $location -SecurityRules $allowRdpRule  #Create NSG with the allow rdp rule

      $nic.NetworkSecurityGroup = $nsg
      $nic | Set-AzNetworkInterface  #Connect the NSG to NIC. We could do the same thing with subnet as well

      Get-AzResource -ResourceGroupName $resourceGroupName | format-table # to get all the resources we have created so far!
      </code>
</pre>

### Image

Determines which operating system and applications are pre-installed. Azure offers many pre built images which can be consumed directly. We can also use market place to search for a VM provided by third party sellers. This is one of the cost deciding factor too. To know which Image to use for creating the VM use below APIs
1. Use the Get-AzVMImagePublisher command to return a list of image publishers:
1. Use the Get-AzVMImageOffer to return a list of image offers
1. The Get-AzVMImageSku command will then filter on the publisher and offer name to return a list of image names

Response from each API would provide information that can be used to call subsequent calls. 

### Storage

Virtual machine needs to have a space to store the operating system image and data. When a virtual box is created, operating system will be hosted in C drive and a temporary disk (D: drive) is also provided. We can add more disk capacity depending on our need.

### Create Virtual Machine using Powershell

<pre><code class="powershell">
$cred = Get-Credential #Type the user name and password for accessing VM
New-AzVm -ResourceGroupName $resourceGroupName -Name "sample-vm" -Location $location -VirtualNetworkName "sample-vm-rg-vnet" -SubnetName "fontend-servers" -SecurityGroupName "sample-vm-rg-nsg" PublicIpAddressName "sample-vm-rg-pip" -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" -Credential $cred
</code><pre>

### Create Virtual Machine using ARM Templates
ARM templates are written using JSON notation. ARM templates follow the [schema](https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json). Most simplified version of an ARM template would have following sections.

<pre>
<code class="json">
  {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "properties":{},
       "variables":{},
       "resources":{}
  }
</code>
</pre>

Below is the complete ARM template for creating a typical Virtual Machine. I have tried to pre-fill the values for most of the paraeters in the template.

<pre>
<code class="json">
  {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "properties":{
          "vmUsername": { "type": "string" },
          "vmUserPassword": {"type": "securestring"},
          "windowsOSVersion": {"type": "string", "defaultValue": "2016-Datacenter"},
          "vmSize": {"type": "string", "defaultValue": "Standard_D2_v3"},
          "location": { "type": "string", "defaultValue": "[resourceGroup().location]"}
        },
        "variables":{
          "vmname": "concat('vm-',location, '-', uniquestring(resourceGroup().id))",
          "storageAccountName": "[concat(vmname, 'storage')]",
          "nicName": "[concat(vmname, 'nic')]",
          "addressPrefix": "10.0.0.0/16",
          "subnetName": "[concat(vmname, 'subnet')]",
          "subnetPrefix": "10.0.0.0/24",
          "publicIPAddressName": "[concat(vmname, 'pip')]",
          "virtualNetworkName": "[concat(vmname, 'vnet')]",
          "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
          "networkSecurityGroupName": "[concat(vmname, 'nsg')]"
        },
        "resources":[
          {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2018-11-01",
            "name": "[variables('publicIPAddressName')]",
            "location": "[parameters('location')]",
            "properties": {
              "publicIPAllocationMethod": "Dynamic"
            }
          },
          {
            "type":  "Microsoft.Network/networkSecurityGroups",
            "apiVersion":  "2019-08-01",
            "name":  "[variables('networkSecurityGroupName')]",
            "location":  "[parameters('location')]",
              "properties": {
                "securityRules": [
                  {
                    "name":  "allow-RDP",
                    "properties": {
                      "priority":  1000,
                      "access":  "Allow",
                      "direction":  "Inbound",
                      "destinationPortRange":  "3389",
                      "protocol":  "Tcp",
                      "sourcePortRange":  "*",
                      "sourceAddressPrefix":  "*",
                      "destinationAddressPrefix":  "*"
                    }
                  }
                ]
              }
            },
            {
              "type": "Microsoft.Network/virtualNetworks",
              "apiVersion": "2018-11-01",
              "name": "[variables('virtualNetworkName')]",
              "location": "[parameters('location')]",
              "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
              ],
              "properties": {
                "addressSpace": {
                  "addressPrefixes": [
                    "[variables('addressPrefix')]"
                  ]
                },
                "subnets": [
                  {
                    "name": "[variables('subnetName')]",
                    "properties": {
                      "addressPrefix": "[variables('subnetPrefix')]",
                      "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
                      }
                    }
                  }
                ]
              }
            },
            {
              "type": "Microsoft.Network/networkInterfaces",
              "apiVersion": "2018-11-01",
              "name": "[variables('nicName')]",
              "location": "[parameters('location')]",
              "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
              ],
              "properties": {
                "ipConfigurations": [
                  {
                    "name": "ipconfig1",
                    "properties": {
                      "privateIPAllocationMethod": "Dynamic",
                      "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]"
                      },
                      "subnet": {
                        "id": "[variables('subnetRef')]"
                      }
                    }
                  }
                ]
              }
            },
            {
              "type": "Microsoft.Compute/virtualMachines",
              "apiVersion": "2018-10-01",
              "name": "[variables('vmName')]",
              "location": "[parameters('location')]",
              "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
              ],
              "properties": {
                "hardwareProfile": {
                  "vmSize": "[parameters('vmSize')]"
                },
                "osProfile": {
                  "computerName": "[variables('vmName')]",
                  "adminUsername": "[parameters('adminUsername')]",
                  "adminPassword": "[parameters('adminPassword')]"
                },
                "storageProfile": {
                  "imageReference": {
                    "publisher": "MicrosoftWindowsServer",
                    "offer": "WindowsServer",
                    "sku": "[parameters('windowsOSVersion')]",
                    "version": "latest"
                  },
                  "osDisk": {
                    "createOption": "FromImage"
                  }
                  ]
                },
                "networkProfile": {
                  "networkInterfaces": [
                    {
                      "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
                    }
                  ]
                },
                "diagnosticsProfile": {
                  "bootDiagnostics": {
                    "enabled": false
                  }
                }
              }
            }
        ]
  }
</code>
</pre>


#### References
1. [Tutorial: Create and Manage Windows VMs with Azure PowerShell](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm?WT.mc_id=thomasmaurer-blog-thmaure)
1. Powershell: [Az.Network API Reference](https://docs.microsoft.com/en-us/powershell/module/az.network/?view=azps-4.3.0)
1. ARM Template Reference: Create subnet - [Microsoft.Network/virtualNetworks/subnets](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/virtualNetworks/subnets)
1. ARM Template Reference: Create storage account - [Microsoft.Storage/storageAccounts](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Storage/storageAccounts)
1. ARM Template Reference: Create public IP address - [Microsoft.Network/publicIPAddresses](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/publicIPAddresses)
1. ARM Template Reference: Create network security group - [Microsoft.Network/networkSecurityGroups](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/networkSecurityGroups)
1. ARM Template Reference: Create virtual network - [Microsoft.Network/virtualNetworks](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/virtualNetworks)
1. ARM Template Reference: Create NIC - [Microsoft.Network/networkInterfaces](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/networkInterfaces)
1. ARM Template Reference: Create virtual machine - [Microsoft.Compute/virtualMachines](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Compute/virtualMachines)

<script>hljs.initHighlightingOnLoad();</script>