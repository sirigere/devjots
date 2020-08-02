---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204, PaaS, AppService, WebApp]
identifier: Az204-Compute-AppService
title: Using App Service Plan in Azure 
---
#### Skills Measured
1. Create an Azure App Service Web App
1. Enable diagnostics logging
1. Deploy code to a web app
1. Configure web app settings including SSL, API, and connection strings
1. Implement autoscaling rules, including scheduled autoscaling, and scaling by operational or system metrics

We will do the following to understand different concepts involved to learn the skill set being measured.
1. [Provision an App Service Plan and create a WebApp instance](#provision-an-app-service-plan-and-create-a-webapp-instance)
1. [Deploy the code from a source code repository](#deploy-the-code-from-a-source-code-repository)
1. [Update the code to use SQL Server database](#update-the-code-to-use-sql-server-database)
1. Add rules for auto scaling
1. Add diagnostic logging to the application

#### Provision an App Service Plan and create a WebApp instance
Navigate to [Azure Shell](https://shell.azure.com/) and sign-in. Execute below commands to create an app service and web app

<pre>
  <code>
    randomNumber=$RANDOM 
    resourcegroupName=learnAppservice$randomNumber
    az configure --defaults location=eastus
    az configure --defaults group=$resourcegroupName
    az group create --name $resourcegroupName
    az appservice plan create --name appservice$randomNumber --sku Free
    az webapp create --name webapp$randomNumber -p appService$randomNumber --query defaultHostName
  </code>
</pre>

Above script when run sequentially will create a app service plan and web app.

#### Deploy the code from a source code repository

Create a working directory to host an WebApi application using command `mkdir webapi4appservice`. Make that as the current working directory by navigating to that newly created directory `cd webapi4appservice`. Execute the command `dotnet new webapi` to create a new `WebApi` project.

![New WebApi project](\assets\images\azure_appservice\newwebapiproject.png)

ensure it run by executing the command `dotnet run`

![Run the application](\assets\images\azure_appservice\runtheapplocaly.png)

We have multiple options for deploying this application to Azure. Below picture summarizes the current options.

![App Service deployment options](\assets\images\azure_appservice\deploymentoptions.png)

Below are the steps to be followed for deploying using zipdeploy. Execute the command `dotnet publish -c Release -o out`

![Publish the application](\assets\images\azure_appservice\publishapplication.png)

change the working directory to out by executing the command `cd out`. zip the content of `out` folder using the command `zip -r site.zip *`

![Zip up out directory](\assets\images\azure_appservice\zipfoldercontent.png)

Execute command `az webapp deployment source config-zip --src site.zip` from Azure Shell

![Deploy](\assets\images\azure_appservice\deploytheapp.png)

Verify the application is deployed by browsing to one of the supported api end points. In this case `https://<web app name>.azurewebsites.net/api/Values`

#### Update the code to use SQL Server database

#### References

<script>hljs.initHighlightingOnLoad();</script>