---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204, IaaS]
identifier: Az204-Compute-ARM-VM
title: Provision VM using ARM Templates
---

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
1. ARM Template Reference: Create subnet - [Microsoft.Network/virtualNetworks/subnets](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/virtualNetworks/subnets)
1. ARM Template Reference: Create storage account - [Microsoft.Storage/storageAccounts](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Storage/storageAccounts)
1. ARM Template Reference: Create public IP address - [Microsoft.Network/publicIPAddresses](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/publicIPAddresses)
1. ARM Template Reference: Create network security group - [Microsoft.Network/networkSecurityGroups](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/networkSecurityGroups)
1. ARM Template Reference: Create virtual network - [Microsoft.Network/virtualNetworks](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/virtualNetworks)
1. ARM Template Reference: Create NIC - [Microsoft.Network/networkInterfaces](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Network/networkInterfaces)
1. ARM Template Reference: Create virtual machine - [Microsoft.Compute/virtualMachines](https://docs.microsoft.com/en-us/azure/templates/Microsoft.Compute/virtualMachines)

<script>hljs.initHighlightingOnLoad();</script>