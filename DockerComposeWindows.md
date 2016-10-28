<<<<<<< HEAD
# Part 3 - Deploy a multi-containers application on Windows

In this part, you will learn how to use Docker and Docker Compose to deploy a multi-containers application on Windows.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

## Introduction

The application that you are going to deploy within Windows Containers is pretty simple as it is composed of three components, all developed using [ASP.NET Core 1.0](https://github.com/aspnet):

- Web Front: a simple web application that makes AJAX calls the three other APIs and display their results on the home page
- Products Api : a simple rest API that returns a message with its version and the hostname where it is currently executed
- Ratings Api : a simple rest API that returns a message with its version and the hostname where it is currently executed

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

## Tag and push your .NET Core image to the Docker Hub

To be able to push the image into your Docker Hub account, its name needs to start by your Docker Hub identifier. For example, my identifier is jcorioland, so all images I want to push in the hub should start with jcorioland/IMAGE_NAME.
You can also add tag to an image, to version it and be able to push multiple versions of one container image. Here, we are going to use the .NET Core version to tag the image, using the **docker tag** command:

```
docker tag dotnetcore YOUR_DOCKER_HUB_IDENTIFIER/dotnetcore:1.0.0-preview2-003131
```

If you list the image available on the machine using the **docker images** commands you should see two entries with the same ID, dotnetcore with its default tag "latest" and the one you have just tagged.

Now you can do a **docker login** and enter your Docker Hub credentials once prompted.

![DockerBuild]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerLogin.png)

As soon as you are successfuly logged in you can use the **docker push** command to push your image in your Docker Hub account:

```
docker push YOUR_DOCKER_HUB_IDENTIFIER/dotnetcore:1.0.0-preview2-003131
```

![DockerPush]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerPush.png)

## Build and push images for Products API, Ratings API and Front

In this step, you will build and push the image for each component of the application.
In the Sources directory, there is a folder for each project and each one contains a Dockerfile that allows to package the application. As you can see, these Dockerfiles are not based on the microsoft/windowsservercore:latest image, but on the dotnetcore image that you have built at the step before.

In each Dockerfile, update the image name in the **FROM** section to use your image.

Then, you can build and push each image into your Docker Hub account, using the **docker build**, **docker tag**, **docker login** and **docker push** as explained before.

Once you are done, you should have the 3 images on your machine and in your Docker Hub account:

![DockerPush]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerImages.png)

## Use Docker-Compose to start your application

Docker Compose is a CLI tool that allows to start multiple containers at the same time. In this example, we want to launch 3 containers, one for each component of the application.

If you look at the root of the Sources directory, you will see a docker-compose.yml file. This file defines the different services that will compose the application:

```
version: '2'
networks:
  nat:
    external: true
    
services:
  products-service:
    image: jcorioland/products-api:1.0.0-preview2-003131
    ports:
      - "5001:5001"
    networks:
      nat:
        ipv4_address: 172.26.127.31
  ratings-service:
    image: jcorioland/ratings-api:1.0.0-preview2-003131
    ports:
      - "5002:5002"
    networks:
      nat:
        ipv4_address: 172.26.127.32
  shop-front:
    image: jcorioland/shop:1.0.0-preview2-003131
    ports:
      - "5000:5000"
    networks:
      nat:
        ipv4_address: 172.26.127.30
    environment:
      - SHOP_PRODUCTS_API_URL=http://172.26.127.31:5001
      - SHOP_RATINGS_API_URL=http://172.26.127.32:5002
```

All you need to do here is to replace the image name of each component by the names of your images. And then start the application using the following command:

```
docker-compose up
```

This will create three containers:

![DockerComposeUp]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerComposeUp.png)

Wait a minute to be sure that all applications are started in the containers:

![DockerComposeUp2]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerComposeUp2.png)

And then browse http://172.26.127.30:5000 on the machine you have started the containers:

![ItWorks]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/ItWorks.png)

Et voilà! You have completed the part 3 of this Hands-on-Lab.
=======
# Part 3 - Deploy a multi-containers application on Windows

In this part, you will learn how to use Docker and Docker Compose to deploy a multi-containers application on Windows.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

The application that you are going to deploy within Windows Containers is pretty simple as show in the figure above:

![MyShopSample]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/MyShopSample.png)

The content of this part will be available soon.
>>>>>>> master
