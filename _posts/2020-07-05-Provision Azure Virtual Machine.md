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

Planning is essential before creating virtual machine. As some of the configurations cannot be undone once the virtual machine is created. Below are some of the things to be aware of before creating a virtual machine. Please note this is not the complete list.
+ What operating system we plan to use
+ Topology of the network
+ Scalability, availability expectations of the solution
+ What hardware specification we are looking at? Example, amount of RAM, no of CPUs, size and type (SSD, HSD) of hard disk we are looking for
+ How do we plan to access the virtual machine? Is it public facing or only accessible within an existing/new virtual network etc..

## What operating system we plan to use
Depending on the answer, we can decide from where we can get the image of the operating system. Possible options are
+ We already might have an image that we plan on using.
+ Search for the image from Azure market place.
+ Some of the operating systems or the version of the OS may not be supported by the Azure platform itself.

## What hardware specification we are looking at?
+ Azure offers multiple pre configured hardware configurations to choose from
+ Not all VM configurations available in all Azure regions

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
Lets think of a simple virtual network that we would like to create. Below is one such specification
- Virtual Network will be created in eastus location
- Specific IP address identifies an host/ device within a netowrk. For a network, we need to specify a range of IP addresses that are allowed within that network. Common mechanism is to use CIDR notation. Using CIDR notation, lets define our network to have address prefix of 10.10.0.0/16. This would freeze first 16 bits of the address space. 
- Create a subnet with address prefix of 10.10.1.0/24
- Create a dynamically allocated public ip address so that we can easily access our network/ device through internet.
- Create a network interface card which would be use the public ip address created and will be part of the subnet we defined above
- Create a network security group and rule that would allow RDP access. In other words allow inbound requests on port 3389

We would need to work with multiple Powershell commandlets to accomplish this goal. We would use the following commandlets to accomplish the goal.
+ Create a resource group using ```New-AzResourceGroup``` commandlet
+ Create a Virtual Network using ```New-AzVirtualNetwork``` and ```Set-AzVirtualNetwork``` commandlets
+ Create subnet using ```Add-AzVirtualNetworkSubnetConfig``` commandlet
+ Create a Network Security Group allowing access to RDP using ```New-AzNetworkSecurityGroup``` and ```New-AzNetworkSecurityRuleConfig``` commandlets 
+ Create a Network Interface Card using ```New-AzNetworkInterface``` and ```Set-AzNetworkInterface``` commandlets
+ Create public IP Address using ```New-AzPublicIpAddress``` commandlet
+ I have omitted the commandlets that would be required to authenticate and authorize the session as that would be repeatation or is not the main theme of this post. 

<pre>
    <code class="powershell">
      $resourcePrefix  = "demo-vm-creation-"
      $resourceGroupName = $resourcePrefix + "rg"
      $vnetName = $resourcePrefix + "vnet"
      $subnetName = $resourcePrefix + "frontend-subnet"
      $nicName = $resourcePrefix + "nic"
      $ipName = $resourcePrefix + "ip"
      $location = "eastus"

      Connect-AzAccount   #Connect to your Azure account and subscription

      New-AzResourceGroup -Name $resourceGroupName -Location $location   #Creates a Resource Group at eastus region

      #Create an in-memory VNet configuration object
      $vnet = New-AzVirtualNetwork -Name $vnetName -Location $location  -ResourceGroupName $resourceGroupName -AddressPrefix "10.10.0.0/16"    

      Add-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet -AddressPrefix "10.10.1.0/24"   #Add subnet configuration to the virtual network

      $vnet | Set-AzVirtualNetwork #Create the virtual network

      $vnet = Get-AzVirtualNetwork -Name $vnetName #Re-initialize the vnet variable as we need the id of the newly added subnet

      $subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet #Get the definition of subnet as we need this information in next steps

      $pip = New-AzPublicIpAddress -Name $ipName -ResourceGroupName $resourceGroupName -Location $location -AllocationMethod Dynamic  #Create a public ip address

      $nic = New-AzNetworkInterface -Name $nicName -Location $location -ResourceGroupName $resourceGroupName -Subnet $subnet -PublicIpAddress $pip

      $allowRdpRule = New-AzNetworkSecurityRuleConfig -Name "Allow RDP" -Protocol Tcp -Direction Inbound -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389

      $nsg = New-AzNetworkSecurityGroup -Name $resourcePrefix + "nsg" -ResourceGroupName $resourceGroupName -Location $location -SecurityRules $allowRdpRule  #Create NSG with the allow rdp rule

      $nic.NetworkSecurityGroup = $nsg
      $nic | Set-AzNetworkInterface  #Connect the NSG to NIC. We could apply this nsg at subnet level as well

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
