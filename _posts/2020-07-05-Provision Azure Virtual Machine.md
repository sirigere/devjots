---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204, IaaS]
identifier: Az204-Compute-VM
title: Provision VM and Configure for remote access
---

``Azure Virtual Machine`` is an ``IaaS`` (Infrastructure as a Service) offering from Azure. Every Virtual Machine is composed of multiple virtual resources. Some of them are independent and some of them are dependent. In other words, we could use these existing resources and create an virtual machine. In this post, I am going to discuss these independent virtual services and how to provision them. There are many ways we can provision these Azure Virtual Machine. We will familiarize ourselves with how we can create these resources using ``PowerShell``, ``Azure CLI`` and ``ARM`` templates. Below are the main components of an Azure Virtual Machine.

+ Networking
+ Storage
+ Image

## Networking
Below hierarchical list shows how different components are dependent on each other.

+ Virtual Network (VNet)
  + Subnet
    + Network Security Group (NSG). This is optional to create.
       + Network Security Group Rule
    + Network Interface (NIC)
      + Network Security Group (NSG)
        + Network Security Group Rule
      + Public Ip Address

#### Powershell script to create networking resources
Below script does the following
+ Create a resource group using ```New-AzResourceGroup``` commandlet
+ Create a Virtual Network using ```New-AzVirtualNetwork``` and ```Set-AzVirtualNetwork``` commandlets
+ Create subnet using ```Add-AzVirtualNetworkSubnetConfig``` commandlet
+ Create a Network Security Group allowing access to RDP using```New-AzNetworkSecurityGroup``` and ```New-AzNetworkSecurityRuleConfig``` commandlets 
+ Create a Network Interface Card using```New-AzNetworkInterface``` and ```Set-AzNetworkInterface``` commandlets
+ Create public IP Address using```New-AzPublicIpAddress``` commandlet

<pre>
    <code class="powershell">
      $resourceGroupName = "sample-vm-rg"
      $location = "eastus"

      Connect-AzAccount   #Connect to your Azure account

      New-AzResourceGroup -Name $resourceGroupName -Location $location   #Create Resource Group

      $vnet = New-AzVirtualNetwork -Name "sample-vm-rg-vnet" -Location $location  -ResourceGroupName $resourceGroupName -AddressPrefix "10.0.0.0/16"    #Create VNet

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

1. Use the ```Get-AzVMImagePublisher``` command to return a list of image publishers:
1. Use the command ```Get-AzVMImageOffer``` to return a list of image offers
1. The ```Get-AzVMImageSku``` command will then filter on the publisher and offer name to return a list of image names

Response from each API would provide information that can be used to call subsequent calls. 

### Storage

Virtual machine needs to have a space to store the operating system image and data. When a virtual box is created, operating system will be hosted in C drive and a temporary disk (D: drive) is also provided. We can add more disk capacity depending on our need.

### Create Virtual Machine using Powershell

<pre><code class="powershell">
$cred = Get-Credential #Type the user name and password for accessing VM
New-AzVm -ResourceGroupName $resourceGroupName -Name "sample-vm" -Location $location -VirtualNetworkName "sample-vm-rg-vnet" -SubnetName "fontend-servers" -SecurityGroupName "sample-vm-rg-nsg" PublicIpAddressName "sample-vm-rg-pip" -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" -Credential $cred
</code></pre>

#### References
1. [Tutorial: Create and Manage Windows VMs with Azure PowerShell](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm?WT.mc_id=thomasmaurer-blog-thmaure)
1. Powershell: [Az.Network API Reference](https://docs.microsoft.com/en-us/powershell/module/az.network/?view=azps-4.3.0)

<script>hljs.initHighlightingOnLoad();</script>
