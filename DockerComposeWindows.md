# Part 3 - Deploy a multi-containers application on Windows

In this part, you will learn how to use Docker and Docker Compose to deploy a multi-containers application on Windows.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

## Introduction

The application that you are going to deploy within Windows Containers is pretty simple as it is composed of three components, all developed using [ASP.NET Core 1.0](https://github.com/aspnet):

- Web Front: a simple web application that makes AJAX calls the three other APIs and display their results on the home page
- Products Api : a simple rest API that returns a message with its version and the hostname where it is currently executed
- Ratings Api : a simple rest API that returns a message with its version and the hostname where it is currently executed

All the source code for this application can be found in the [Sources](https://github.com/jcorioland/WindowsContainersHOL/tree/master/Sources) folder of this repository.
You can use any text editor to visualize the sources or you can install [Visual Studio Code](https://code.visualstudio.com) (available on Linux, macOS or Windows) to be able to work with this source code.

First, clone this repository using the following command:

```
git clone https://github.com/jcorioland/WindowsContainersHOL.git
```

The project is quite simple:

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

As there are two different images for Windows Server Containers, you can choose to build your images based on Windows Server Core or Nano Server (which is smaller). In each project's folder, you will see two Dockerfile, one for Windows Server Core and one for Nano Server.

Open a PowerShell session and go to the ProductsApi directory. Then you can build the products API image using:

```
docker build -t products-api -f Dockerfile.Nano .
```

Then, go to the RatingsApi folder and build the ratings API image:

```
docker build -t ratings-api -f Dockerfile.Nano .
```

And finally, do the same for the web front, in the ShopFront directory:

```
docker build -t shop-front -f Dockerfile.Nano .
```

Note : if you want build image based on Windows Server Core, just use the Dockerfile.WSCore instead of the Dockerfile.Nano.

## Create a Docker Hub account

If you do not already have a Docker Hub account, please go to https://hub.docker.com and create a new one, it's free!

## Tag and push your images to the Docker Hub

To be able to push the image into your Docker Hub account, its name needs to start by your Docker Hub identifier. For example, my identifier is jcorioland, so all images I want to push in the hub should start with jcorioland/IMAGE_NAME.
You can also add tag to an image, to version it and be able to push multiple versions of one container image.

You can use the **docker tag** command to add a tag to the images you have built in the previous steps.

```
docker tag products-api YOUR_DOCKER_HUB_IDENTIFIER/products-api:1.0.0-preview2-nanoserver
```

Repeat this operation for all the images.

Now you can do a **docker login** and enter your Docker Hub credentials once prompted.

![DockerBuild]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerLogin.PNG)

As soon as you are successfuly logged in you can use the **docker push** command to push each images in your Docker Hub account:

```
docker push YOUR_DOCKER_HUB_IDENTIFIER/products-api:1.0.0-preview2-nanoserver
```

![DockerPush]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerPush.PNG)

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
    image: jcorioland/products-api:1.0.0-preview2-nanoserver
    ports:
      - "5001:5001"
    networks:
      nat:
        ipv4_address: 172.26.127.31
  ratings-service:
    image: jcorioland/ratings-api:1.0.0-preview2-nanoserver
    ports:
      - "5002:5002"
    networks:
      nat:
        ipv4_address: 172.26.127.32
  shop-front:
    image: jcorioland/shop-front:1.0.0-preview2-nanoserver
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
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerComposeUp.PNG)

Wait a minute to be sure that all applications are started in the containers:

![DockerComposeUp2]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/DockerComposeUp2.PNG)

And then browse http://172.26.127.30:5000 on the machine you have started the containers:

![ItWorks]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/Part3/ItWorks.PNG)

Et voilà! You have completed the part 3 of this Hands-on-Lab.
