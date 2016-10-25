# Part 1 - Setup your environment

In this part, you will learn how to setup Windows Server 2016 or Windows 10 to work with Windows Containers and Docker.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

## Introduction

Windows Containers are available on Windows Server 2016 and Windows 10 Anniversary Update. There are several ways to get your environment ready.

## Windows 10 Anniversary Update

The simple way to install and configure Windows Container features on your Windows 10 Anniversary Update machine is to download [Docker for Windows](https://docs.docker.com/docker-for-windows/). You need to work with the beta channel to get the support of Windows Container with Docker for Windows.

If you do not want use Docker for Windows, open an elevated PowerShell session and follow these steps: 

1. Enable the containers feature on your machine:

```PowerShell
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

2. Enable the Hyper-V feature on your machine:

```PowerShell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

3. Restart your computer:

```PowerShell
Restart-Computer -Force
```
Once restarted, re-open an elevated PowerShell session and install Docker using the following commands. 

4. Download Docker:

```PowerShell
Invoke-WebRequest “https://master.dockerproject.org/windows/amd64/docker-1.13.0-dev.zip” -OutFile “$env:TEMP\docker-1.13.0-dev.zip” -UseBasicParsing
```

5. Expand the Docker archive:

```PowerShell
Expand-Archive -Path “$env:TEMP\docker-1.13.0-dev.zip” -DestinationPath $env:ProgramFiles
```

6. Add the Docker directory in system path:

```PowerShell
[Environment]::SetEnvironmentVariable(“Path”, $env:Path + “;C:\Program Files\Docker”, [EnvironmentVariableTarget]::Machine)
```

7. Register the Docker deamon as a service on your machine:

```PowerShell
dockerd –register-service
```

8. Start the Docker service:

```PowerShell
Start-Service Docker
```

After this step, you should be able to work with Docker on your machine. You can type docker info to check that all is good. 

![PSResult]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironment1.jpg

## Windows Server 2016

Docker for Windows does not work on Windows Server 2016, so you need to install the containers feature by yourself, but it’s really simple. 

Open an elevated PowerShell session and follow these steps: 

1. Install the containers feature

```PowerShell
Install-WindowsFeature containers
```

2. Restart the machine

```PowerShell
Restart-Computer -Force
```

Once restarted, re-open an elevated PowerShell session and install Docker. 

3. Download the Docker engine

```PowerShell
Invoke-WebRequest “https://download.docker.com/components/engine/windows-server/cs-1.12/docker-1.12.2.zip” -OutFile “$env:TEMP\docker.zip” -UseBasicParsing
```

4. Install the Docker engine

```PowerShell
Expand-Archive -Path “$env:TEMP\docker.zip” -DestinationPath $env:ProgramFiles
```

5. Add the Docker directory in the system path

```PowerShell
[Environment]::SetEnvironmentVariable(“Path”, $env:Path + “;C:\Program Files\Docker”, [EnvironmentVariableTarget]::Machine)
```

6. Register the Docker deamon as a service

```PowerShell
dockerd.exe –register-service
```

7. Start the Docker service
```PowerShell
Start-Service Docker
```

After this step, you should be able to work with Docker on your machine. You can type **docker info** to check that all is good.

## Windows Server 2016 with Containers on Azure

If you do not want install and configure yourself Windows Server 2016 and the containers features, the best and quick way to get started is to use the Windows Server 2016 Datacenter with Containers image available in the Microsoft Azure Marketplace. You don’t have a Microsoft Azure account yet? Create one and try Azure for free now from [http://aka.ms/tryazure](http://aka.ms/tryazure).

Sign in to your Azure account on [http://portal.azure.com](http://portal.azure.com) and search for Windows Server 2016. In the list, you will find the image “with Containers”:

![Step1]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure1.png

Click on this image in the results list:

![Step2]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure2.jpg

Then click on Create to start the creation wizard:

![Step3]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure3.png

Configure the basic settings of your new virtual machine: name, type of disk, username, password and the region where you want to deploy this machine. 

![Step4]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure4.jpg

Then click OK and choose the size of the virtual machine. Click OK when you have made your choice. 
In the next step, the only mandatory option is the network. Click on the Virtual Network option and click Create New: 

![Step5]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure5.png

Give a name to your network and the address space / range you want to work with. Click OK to validate. 
Now, you should be able to validate the step 3 and click OK to start the deployment once the validation step completed:

![Step6]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure6.jpg

Wait for the deployment to be completed: 

![Step7]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure7.jpg

Once deployed, click on the “Connect” button in the virtual machine general overview:

![Step8]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure8.png

It will launch an RDP session. Use your credentials to start a session. Open PowerShell and start the docker service using the following command:

```PowerShell
Start-Service Docker
```

Once started, type **docker info** to validate that all is working as expected. 

![Step9]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure9.jpg

## Pull the base container Images

Now that you have a machine configured to work with Docker and Windows Containers you need to pull a base image. 
There are two base images available: [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/) and [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/). 

```PowerShell
docker pull microsoft/windowsservercore
```

```PowerShell
docker pull microsoft/nanoserver
```

NB: if you have deployed the pre-configured virtual machine in Azure, the two base images are already pulled.

Once completed, you can type docker image to check that the image is available on your machine: 

![Step10]:https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/SetupEnvironmentAzure10.jpg

Your environment is now configured. You can go to the [Part 2](https://github.com/jcorioland/WindowsContainersHOL/blob/master/HelloWindowsContainers.md) that will help you to discover Windows Containers!
