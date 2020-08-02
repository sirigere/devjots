---
layout: post
author: Sanjeev
comments: true
categories: [Certification]
tags: [Certification, AZ-204, IaaS, Containers, Docker]
identifier: Az204-Compute-Container-Image
title: Using Containers in Azure 
---
#### Skills Measured
1. Create container images for solutions by using Docker
1. Publish an image to the Azure Container Registry
1. Run containers by using Azure Container Instance

We will do the following to understand different concepts involved to learn the skill set being measured. Below code assumes windows host machine with docker desktop installed. 
+ [Create and publish a .Net Core WebApi projectlink](#create-and-publish-a-net-core-webapi-project)
+ [Package and Push the app as a docker Image to dockerhub](#package-and-push-the-app-as-a-docker-image-to-dockerhub)
+ [Create an Azure WebApp and deploy the image](#create-an-azure-webapp-and-deploy-the-image)
+ [Create and push the docker image to Azure container registry](#create-and-push-the-docker-image-to-azure-container-registry)
+ [Deploy the image from Azure Container Registry to WebApp created earlier](#deploy-the-image-from-azure-container-registry-to-webapp-created-earlier)
+ [Create and deploy the image from Azure Container Registry to Azure Container Instance](#create-and-deploy-the-image-from-azure-container-registry-to-azure-container-instance)

#### Create and publish a .Net Core WebApi project
Run the below code from a command line window

`mkdir webapis` Create a working directory to host the project

`cd webapis` make that directory as the working directory
  
Run the command `dotnet new webapi` to scaffold a new ASP.NET WebAPI Project

![Create webapi result](\assets\images\azure_containers\createwebapiproject.png)

Execute `dotnet run` to run the application

![run webapi project locally](\assets\images\azure_containers\runwebapilocally.png)

Application is now running on ports 5000 (http) and 5001 (https). Confirm that the application is running by browsing to url `https://localhost:5001/WeatherForecast`

![webapi output in browser](\assets\images\azure_containers\webappoutput.png)

#### Package and Push the app as a docker Image to dockerhub
publish the application to out directory using command `dotnet publish -c Release -o out`

![webapi project published](\assets\images\azure_containers\projectpublished.png)

Within the project directory, create a file with name `Dockerfile` without any extension. Use the command `echo FROM > Dockerfile` to create a new file. Copy the below content in that file.

<pre>
  <code>
    FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
    # Use the base image of ASP.Net 3.1
    WORKDIR /app
    # Set app as the working directory
    COPY /out .
    # Copy the content from the published directory
    EXPOSE 80
    EXPOSE 443
    # Expose ports 80 (http) and 444 (https)
    ENTRYPOINT ["dotnet", "webapis.dll"]
    # Starting command for this image
  </code>
</pre>

Run the command `docker build -f Dockerfile -t webapis:v1` to build the docker image of the application with tag `v1`

![Build docker image of the application](\assets\images\azure_containers\builddockerimage.png)

Run the command `docker images` to make sure you have the container in the local repo

![Verifiy the image in local repo](\assets\images\azure_containers\dockerimageinlocalrepo.png)

Run the image using the command `docker run --rm -it -p 5000:80 -p 5001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ webapis:v1` and run command `docker ps` to check if the container is running.

![Run the docker image](\assets\images\azure_containers\rundockerimage.png)

Navigating to the url `https://localhost:5001/WeatherForecast` should give the similar out as before

![webapi output in browser](\assets\images\azure_containers\webappoutput.png)

Execure command `docker tag webapis:v1 <docker account name>/webapis:v1` in preparationg for moving the image to dockerhub. Run command `docker push <docker account name>/webapis:v1`

![webapi output in browser](\assets\images\azure_containers\pushimagetodhub.png)

#### Create an Azure WebApp and deploy the image
Navigate to [Azure Shell](https://shell.azure.com/) and sign-in. 

<pre>
  <code>
    let randomNum=$RANDOM*$RANDOM
    # Generate a random number to use so that we do not have to revisit the names used in the sample
    resourceGroupName="lab-containers"$randomNum"-Group"
    # Resource group name
    az configure --defaults location=eastus
    az configure --defaults group=$resourceGroupName
    # We do not have to pass the location and resource group attribute once configured so the length of the command becomes small
    az group create --name $resourceGroupName
    #Create the resource group
    appServiceName="appService"$randomeNum
    az appservice plan create --name $appServiceName --sku Free --is-linux
    #Create a linux free app service plan
    az webapp create --name webapp$randomNum -p $appServiceName -i &lt;docker account name&gt;/webapis:v1
    #Creates a web application using the docker image. Replace the docker account name in the above command.
    az webapp show --name webapp$randomNum --query enabledHostNames[0]
    # Execute to get the url to the website
  </code>
</pre>

Navigating to the <url returned>/WeatherForecast, should get the similar response from the API

![API response from Azure webapp](\assets\images\azure_containers\apiresponseinazure.png)

#### Create and push the docker image to Azure container registry

<pre>
  <code>
    #Create an instance of Azure Container Registry with admin user
    az acr create --name acr$randomNum --sku Basic --admin-enabled --query loginServer
    #Above command returns the login server uri. Execute below command will give the password for the admin user
    az acr credential show --name acr$randomNum --query passwords[0].value    
  </code>
</pre>

From your local machine, execute below docker commands `docker login <acr login server>.azurecr.io ` to upload the image to acr

![Login to ACR](\assets\images\azure_containers\logintoacr.png)

Tag the image for acr by executing the command `docker tag webapis:v1 <acr login server>.azurecr.io/webapis:v1` and push the image to the registry by executing the command `docker push <acr login server>.azurecr.io/webapis:v1`

![Push image to ACR](\assets\images\azure_containers\pushimagetoacr.png)

#### Deploy the image from Azure Container Registry to WebApp created earlier
Lets make some change to the code so that we are sure we are deploying the new image only available from ACR image registry. Update the `WeatherForecast` class with new attributing ReportingTime as `public DateTime ReportingTime => DateTime.Now;`. This would provide when the weather was reported.Execute below commands to build and push the image to acr.

<pre>
  <code>
    dotnet build
    dotnet publish -c Release -o out
    docker build -f Dockerfile -t webapis:v2 .
    docker tag webapis:v2 &lt;acr login server&gt;.azurecr.io/webapis:v2
    docker push &lt;acr login server&gt;.azurecr.io/webapis:v2
    az webapp config container set --name webapp$randomNum --docker-custom-image-name acr$randomNum.azurecr.io/webapis:v2 --docker-registry-server-url https://acr$randomNum.azurecr.io --docker-registry-server-user acr$randomNum --docker-registry-server-password &lt;Password for admin user in acr &gt;
  </code>
</pre>

When navigate to `https://<Webapp url>/WeatherForecast` we should see the updated response

![Response with Reported time](\assets\images\azure_containers\updatedimageonwebapp.png)

#### Create and deploy the image from Azure Container Registry to Azure Container Instance


#### References
* [Docker Documentation](https://docs.docker.com/) 
* [Host local containerized website in https](https://github.com/dotnet/dotnet-docker/blob/master/samples/host-aspnetcore-https.md)
* [Microsoft .Net Images in DockerHub](https://hub.docker.com/_/microsoft-dotnet-core)
* [Run a custom Docker image in App Service](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image)

<script>hljs.initHighlightingOnLoad();</script>