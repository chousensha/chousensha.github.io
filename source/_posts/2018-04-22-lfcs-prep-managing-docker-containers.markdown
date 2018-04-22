---
layout: post
title: "LFCS prep - Managing Docker containers"
date: 2018-04-22 07:30:54 -0400
comments: true
categories: [linux, sysadmin]
keywords: docker, docker containers, containers, docker tutorial, docker centos, lfcs, rhcsa, lfcsa
description: Using Docker containers on CentOS
---

In today's post we'll look at Docker containers. Containers are lightweight application virtualization platforms that are isolated, but contrary to virtual machines, they share resources with the host OS. They are faster and less resource intensive than traditional VMs.

<!-- more -->

I've had errors with the straightforward approach of installing Docker on CentOS7. So I installed by following the steps mentioned in the [article about getting Docker Community Edition on CentOS](https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository)

Install needed packages:

```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

Set up the stable Docker repository:

```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

Install Docker CE:

```
yum install docker-ce
```

If you get errors, look in the journal log and troubleshoot from there, but this approach solved the problems for me:

```
journalctl -u docker.service
```

Start the Docker service:

```
systemctl start docker
```

Check information about your Docker installation:

```
docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.03.0-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: cfd04396dc68220d1cecbe686a6cc3aa5ce3667c
runc version: 4fc53a81fb7c994640722ac585fa9ca548971871
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-693.17.1.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 4
Total Memory: 1.687GiB
Name: CentOS7
ID: JRI7:OB26:2HEE:2K4Y:OCOZ:I376:F27U:22C4:L3BU:VPYQ:XZO3:FPEW
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
```

Get more details about the version:

```
docker version
Client:
 Version:	18.03.0-ce
 API version:	1.37
 Go version:	go1.9.4
 Git commit:	0520e24
 Built:	Wed Mar 21 23:09:15 2018
 OS/Arch:	linux/amd64
 Experimental:	false
 Orchestrator:	swarm

Server:
 Engine:
  Version:	18.03.0-ce
  API version:	1.37 (minimum version 1.12)
  Go version:	go1.9.4
  Git commit:	0520e24
  Built:	Wed Mar 21 23:13:03 2018
  OS/Arch:	linux/amd64
  Experimental:	false
```

The docker CLI has a lot of commands:

```
docker --help

Usage:	docker COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

There are tons of containers available at https://hub.docker.com/explore/ . Here, let's look for a hello world one:

```
docker search hello
NAME                                       DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
hello-world                                Hello World! (an example of minimal Dockeriz…   495                 [OK]
```

Pull the image:

```
docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest
```

Run the demo image:

```
docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

The Docker default storage is at <code>/var/lib/docker</code>.  Now let's go beyond the basics and get an Ubuntu image:

```
docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
d3938036b19c: Pull complete
a9b30c108bda: Pull complete
67de21feec18: Pull complete
817da545be2b: Pull complete
d967c497ce23: Pull complete
Digest: sha256:9ee3b83bcaa383e5e3b657f042f4034c92cdd50c03f73166c145c9ceaea9ba7c
Status: Downloaded newer image for ubuntu:latest
```

Check available images:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              c9d990395902        9 days ago          113MB
hello-world         latest              e38bc07ac18e        10 days ago         1.85kB
```

Run a command inside the Ubuntu container:

```
docker run ubuntu uname -a
Linux 05f2f247622a 3.10.0-693.17.1.el7.x86_64 #1 SMP Thu Jan 25 20:13:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

Check all containers:

```
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
05f2f247622a        ubuntu              "uname -a"          About a minute ago   Exited (0) About a minute ago                       naughty_fermi
db7d3fcd06a7        hello-world         "/hello"            10 minutes ago       Exited (0) 9 minutes ago                            trusting_mirzakh
```

Take note of the container ID if you want to start a specific container:

```
docker start 05f2f247622a
05f2f247622a
```

To start an interactive session in the container with an allocated TTY, do the following:

```
docker run -it ubuntu bash
root@c8561feac38d:/# cat /etc/issue
Ubuntu 16.04.4 LTS \n \l
```

Here we've run the bash shell inside the container. To return to the host, type *exit*

We've only scratched the surface, but we can see how powerful and versatile Docker containers can be.

``` 
 _________________________________________
/ Be free and open and breezy! Enjoy!     \
| Things won't get any better so get used |
\ to it.                                  /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
