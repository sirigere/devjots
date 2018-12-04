---
layout: post
author: Sanjeev
comments: true
categories: [Azure]
tags: [ARM]
identifier: ARMTemplates
title: Azure ARM Templates
---
##### Introduction
ARM template stands for Azure Resource Manager template. ARM templates allow you to 
* Define your Azure infrastructure and it's dependencies declaratively.
* Manage Azure resources during your deployment cycle.
* Repeatedly deploy the same infrastructure with confidence
* Accelerating provisioning or deployment rapidly

This is one of the most valuable technology to learn if you are working on Azure platform. ARM templates are the way to go option if you are looking for Infrastructure as Code (IaC) option on Azure.

##### Getting Started

###### Step 1: Gather required Azure resources and dependencies
Using ARM templates we can describe the specification of Azure resources we are interested in. An example of a resource is virtual machine. There are many resources Azure offers. Once we gather information then, we should identify their dependencies. You can request these resources to be provisioned by Azure's resource providers. Resource providers are the services that supplies the Azure resources to deploy and manage. Each resource provider offers operations for working with the resources that are deployed. This link provides the [list](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-supported-services) of all Azure resource providers. 

###### Step 2: Identify the Resource Providers to use for creating Azure resource
This is where the initial confusion starts. How do we know which resource provider to use for the resource I am interested in. There are multiple options to choose from. Depending on your comfortable way to identify them.
* Use Azure portal to create the resource and identify the resource providers
* Refer to a sample ARM template already created and identify the resource provider
* Go through the documentation of resource provider to identify which provider to use

Here are few points to look for
* Is the resource provider available in the region you are looking for
* Is the resource provider registered with your Azure subscription
* API versions for a resource type

##### Incremental deployment of azure templates

For defining LogicApps, Microsoft re-uses the familiar ARM templating constructs. This makes it very interesting! However there are differences. During ARM template deployment, we can specify the mode of deployment. Possible values are Full, Incremental, or Validate. One of the interesting mode is incremental deployment. This is very helpful for deployment of LogicApps. In this mode of deployment, only resources identified within the templates are deployed. Where as in Full mode if any resources are missing in the template, such resources are removed as well. It would be helpful to know that incremental mode of deployment may be encouraging and options on table for deployment. In incremental mode of deployment, some of the core attributes/properties of a resources cannot be changed. For example if I had AppService running on F1 SKU, deployment in S1 SKU will not upgrade the resource. At the same time I never saw any error/warning indicator stating that this has not happened.