# Part 3 - Deploy a multi-containers application on Windows

In this part, you will learn how to use Docker and Docker Compose to deploy a multi-containers application on Windows.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

## Introduction

The application that you are going to deploy within Windows Containers is pretty simple as show in the figure above:

![MyShopSample]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/MyShopSample.png)

It is composed of four components, all developed using [ASP.NET Core 1.0](https://github.com/aspnet):

- Web Front: a simple web application that makes AJAX calls the three other APIs and display their results on the home page
- Products Api : a simple rest API that returns a message with its version and the hostname where it is currently executed
- Ratings Api : a simple rest API that returns a message with its version and the hostname where it is currently executed
- Recommandations Api : a simple rest API that returns a message with its version and the hostname where it is currently executed

All the source code for this application can be found in the [Sources](https://github.com/jcorioland/WindowsContainersHOL/tree/master/Sources) folder of this repository.

You can install [Visual Studio Code](https://code.visualstudio.com) (available on Linux, macOS or Windows) to be able to work with this source code.

First, clone this repository using the following command:

```
git clone https://github.com/jcorioland/WindowsContainersHOL.git
```

Then, go to the directory where you cloned the repository and type **code .** to open it with Visual Studio Code:

![VSCode]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/VSCode.png)

The project is quite simple, as you can see on the picture below:

![VSCodeProject]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/VSCodeProject.png)

- The DotnetDocker folder contains a Dockerfile that allows to create a Windows container image with .NET Core installed inside
- The ProductsApi folder contains the code for the Products web Api
- The RatingsApi folder contains the code for the Ratings web Api
- The RecommandationsApi folder contains the code for the Recommandations web Api
- The ShopFront folder contains the code for the web application

## Build your first Docker image

When working with Docker on Linux, you can create a container from an image that has been built by someone and that has been pushed into the [Docker Hub](hub.docker.com/) or in a [Docker Trusted Registry](https://docs.docker.com/docker-trusted-registry/), for private applications. It is exactly the same for Windows Containers image with Docker.

You can also [build your own image](https://docs.docker.com/engine/tutorials/dockerimages/), from an existing container using the **docker commit** command or from a Dockerfile, which is like a receipt file that contains all the instructions to configure and install the application you want to run in a container.

The basic DevOps workflow when working with Docker is:

1. The developer develops the application
2. The developer creates a Dockerfile that describes how to install the application and its dependencies
3. The developer builds a Docker image from the Dockerfile using the **docker build** command
4. The developer pushes the image into a registry, public or private
5. The ops guy pull the image from the registry
6. The ops guy run the container from the image
7. The ops guy operate / monitor the application

If you look at the [Dockerfile in the DotnetDocker](https://github.com/jcorioland/WindowsContainersHOL/blob/master/DotnetDocker/Dockerfile) folder, you will see the following:

```
FROM microsoft/windowsservercore:latest

MAINTAINER Julien Corioland, Microsoft (@jcorioland)

RUN powershell -Command \
        Invoke-WebRequest 'https://dotnetcli.blob.core.windows.net/dotnet/preview/Binaries/1.0.0-preview2-003131/dotnet-dev-win-x64.1.0.0-preview2-003131.zip' -OutFile dotnet.zip; \
        Expand-Archive dotnet.zip -DestinationPath '%ProgramFiles%\dotnet'; \
        Remove-Item -Force dotnet.zip

RUN setx /M PATH "%PATH%;%ProgramFiles%\dotnet"

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip
RUN mkdir warmup \
    && cd warmup \
    && dotnet new \
    && cd .. \
    && rmdir /q/s warmup
```

As you can see, there is a mix of meta commands like FROM, MAINTAINER, RUN (for the complete list, see the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)) that allows to define each operation that should be done to create the container image.
In this case, we use the **microsoft/windowsservercore:latest** base image from the Docker Hub, then we run a Powershell command to download and extract .NET Core, we set an environment variable, and finally we create a first .NET Core project to warm up the SDK.

To build this image, open a PowerShell session in the DotnetDocker directory and use the following command:

```
docker build -t dotnetcore .
```

Wait for the command to complete and use **docker images** to list the image available on your machine. You should see the dotnetcore image that you have just built.

![DockerBuild]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerBuild.png)

## Create a Docker Hub account

If you do not already have a Docker Hub account, please go to https://hub.docker.com and create a new one, it's free!

## Tag and push your image to the Docker Hub