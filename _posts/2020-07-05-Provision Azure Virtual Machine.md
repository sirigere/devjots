---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204]
identifier: Az204-Compute-VM
title: Provision VMs
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

#### References
1. [Powershell Az.Network API Reference](https://docs.microsoft.com/en-us/powershell/module/az.network/?view=azps-4.3.0)

<script>hljs.initHighlightingOnLoad();</script>