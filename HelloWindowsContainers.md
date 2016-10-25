# Part 2 - Hello Windows Containers !

In this part, you will use the Docker command line to run basic operations like running and managing Windows Containers.
[Go back to the HOL home page.](https://github.com/jcorioland/WindowsContainersHOL)

In the [Part 1](https://github.com/jcorioland/WindowsContainersHOL/blob/master/SetupEnvironment.md) you have learned how to configure your environment to be able to work with Windows Containers.

Windows Containers can be used from the [Docker CLI](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiesdTjxPXPAhXrLsAKHQY3DjQQFggeMAA&url=https%3A%2F%2Fdocs.docker.com%2Fengine%2Freference%2Fcommandline%2Fcli%2F&usg=AFQjCNFUtcaiFc4UMmGUNXGn4cPMy4cCmg&sig2=15m0mF2jKo0KvqnT1LzwiA). To get started, open an elevated PowerShell session on the machine you have configured in the previous step and type the following command:

```PowerShell
docker run -it microsoft/windowsservercore cmd
```

This command will start a new container using the Windows Server Core base image and start cmd.exe within the container. The -it option allows to start the container in interactive mode so you can be connected within the container and use CMD from the inside: 

![HelloWindowsContainers1]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/HelloWindowsContainers1.jpg)

If you start another PowerShell window and type **docker ps** you will see that a container is running:

![HelloWindowsContainers2]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/HelloWindowsContainers2.jpg)

Depending on whether you are running the previous command on Windows Server 2016 or Windows 10, you are not working with the same kind of Windows Containers.

Actually, there are two types of Windows Containers:

- Windows Server Containers, that are similar than Linux containers in the concepts
- Hyper-V Containers, that runs inside a small virtual machine on Hyper-V

![HelloWindowsContainers3]
(https://github.com/jcorioland/WindowsContainersHOL/blob/master/Images/HelloWindowsContainers3.jpg)

Windows Server Containers share the same base OS and kernel. Hyper-V containers have been designed to be more isolated that Windows Server Containers and are running directly on the hypervisor. They are not sharing the guest OS nor the kernel. 
When running on Windows 10, you can only work with Hyper-V containers. 
When running on Windows Server 2016, you can choose between Hyper-V and Windows Server containers. By default, when you use the **docker run** command it will start a Windows Server container, but you can specify that you want to run an Hyper-V container by using the flag **--isolation=hyperv**

Of course, you can use all the commands from the Docker CLI with Windows Containers. For example, you can get the execution logs using the **docker logs** command or get the details about a container using the **docker inspect** command.

All the tools to build, push and pull Docker images are also available with Docker containers on Windows. You will learn to work with these tools in the [Part 3](https://github.com/jcorioland/WindowsContainersHOL/blob/master/DockerComposeWindows.md).