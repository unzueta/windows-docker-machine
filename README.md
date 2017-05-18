# Windows Containers in Docker Toolbox for Windows
This is a fork of https://github.com/StefanScherer/packer-windows

If Hyper-V is not available, the work of StefanScherer is also a viable option for Docker Toolbox for Windows. 
This Vagrant environment creates a Docker Machine to work with Windows containers with Windows Toolbox. 
You can easily switch between Docker for Linux containers and the Windows containers.

Tested environments
  * Windows with Docker Toolbox with Vagrant 1.9.4
    * VirtualBox 5.1.22

#### Before you begin

First you need the Windows Server 2016 VM for your hypervisor. 

1. **packer build** to build a Vagrant base box, it's like a Docker image, but for Vagrant VM's
2. **vagrant up** to create a running VM instance of Windows Server 2016
3. **docker run** to run Windows containers in that Windows VM

Step 1 can be done with these steps:

```PowerShell as Administrator
PS C:\ git clone https://github.com/StefanScherer/packer-windows
PS C:\ cd packer-windows
PS C:\ packer build --only=virtualbox-iso windows_2016_docker.json
PS C:\ vagrant box add windows_2016_docker windows_2016_docker_virtualbox.box
```

## Working on Windows Toolbox

### Create the Docker Machine

Spin up the headless Vagrant box with Windows Server 2016 and Docker CE installed.
It will create the TLS certs and create a `windows` Docker machine for your
`docker-machine` binary on your PC. Remeber to run PowerShell as Administrator.

```PowerShell as Administrator
PS C:\ git clone https://github.com/unzueta/windows-docker-machine
PS C:\ cd windows-docker-machine
```
I you want to have an IP address from the same subnet as the Windows host, you can add a bridged network. to do so, edit the Vagrantfile
add the following line after the line 'config.vm.network :private_network, ip: "192.168.99.90", gateway: "192.168.99.1"':
```
config.vm.network "public_network", bridge: "<physical host interface>"
```
The string identifying the desired interface must exactly match the name of an available interface.

If you want to Display the VirtualBox GUI change the setting of v.gui to true
 ```
 config.vm.provider "virtualbox" do |v|
    v.gui = true
 ```
If you want add additional shared folders (i.e. D:\), add the following line:
```
config.vm.synced_folder "D:\\", "/d_drive", automount: true 
```

Save the file and run 

```PowerShell as Administrator
PS C:\ vagrant up --provider virtualbox
```
This will create a virtual machine in virtualbox called windows-docker-machine-default-XXXXXXXXX

### List your new Docker machine

```PowerShell
PS C:\ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.05.0-ce
windows   -        generic      Running   tcp://192.168.99.90:2376            Unknown       Unable to query docker version: 400 Bad Request: client version 1.15 is too old. Minimum supported API version is 1.24, please upgrade your client to a newer version
```
If you added a Bridged Network, you may have to correct the IPAddress line in the file config.json under c:\users\yourusername\.docker\machines\windows

```
 "IPAddress": "192.168.99.90 192.168.10.211",
```
to:
 "IPAddress": "192.168.99.90",
 
Currently there is [an issue](https://github.com/docker/machine/issues/3943) that the client API version of `docker-machine` is too old. But switch Docker environments works as shown below.

### Switch to Windows containers

```Powershell
PS C:\ docker-machine env windows | iex
```

Now your Docker client talks to the Windows Docker engine:

```PowerShell
PS C:\ docker version
Client:
 Version:      1.13.0-dev
 API version:  1.25
 Go version:   go1.7.3
 Git commit:   11d52b0
 Built:        Thu Oct 20 12:22:33 2016
 OS/Arch:      windows/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Fri May  5 15:36:11 2017
 OS/Arch:      windows/amd64
```

### Switch back to Linux Containers

```Powershell
PS C:\docker-machine env default
```

This will set DOCKER environment with Linux Containers.

```Powershell
PS C:\> docker version
Client:
 Version:      1.13.0-dev
 API version:  1.25
 Go version:   go1.7.3
 Git commit:   11d52b0
 Built:        Thu Oct 20 12:22:33 2016
 OS/Arch:      windows/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 21:43:09 2017
 OS/Arch:      linux/amd64
```

### Mounting volumes from your PC

`C:\d_drive` to map local drive.

```Powershell
PS C:\ docker run -it -v "C:\d_drive:C:\d_drive" microsoft/nanoserver powershell
```

This mounts the user directory through the Windows 2016 VM into the Windows Container.

## Further commands

Here is a list of `docker-machine` commands and the equivalent Vagrant command.

| Docker-machine command | Vagrant equivalent |
|---------|-------|
| `docker-machine create -d xxx windows` | `vagrant up --provider xxx`
| `docker-machine regenerate-certs` | `vagrant provision` |
| `docker-machine stop windows` | `vagrant halt`
| `docker-machine start windows` | `vagrant up`
| `docker-machine ssh windows` | `vagrant rdp`
| `docker-machine rm windows` | `vagrant destroy` |
