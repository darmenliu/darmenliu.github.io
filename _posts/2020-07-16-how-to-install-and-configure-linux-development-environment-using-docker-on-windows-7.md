---
layout: post
title: "how to install and configure linux development environment using docker on windows 7"
author: "Yongqiang Liu"
---

# how to install and configure linux development environment using docker on windows 7

[TOC]

# 1.1 install docker on windows

- Step 1:download the DockerToolbox

Download link:

Docker: https://www.docker.com/products/docker-toolbox

Github: https://github.com/docker/toolbox/releases

Recommend: http://get.daocloud.io/#install-docker-for-mac-windows (download faster)

- Step 2: install step by step

Help link: https://docs.docker.com/toolbox/toolbox_install_windows/

- Step 3: copy boot2docker.iso from path : D:\userdata\yliu\.docker\machine\cache to the path: C:\Users\yliu\.docker\machine\cache, (create folder if not exist)
- Step4: run docker

On your Desktop, find the Docker Toolbox icon

![docker1](https://ithorseman.files.wordpress.com/2018/04/docker1.png)

Click the icon to launch a Docker Toolbox terminal.

 

Or you can use the git bash and go to docker directory(C:\Program Files\Docker Toolbox)

Exexute the script start.sh

![docker2](https://ithorseman.files.wordpress.com/2018/04/docker2.png)

![docker3](https://ithorseman.files.wordpress.com/2018/04/docker3.png)

Now you can use docker, you can test the command like “docker images”

![docker4](https://ithorseman.files.wordpress.com/2018/04/docker4.png)

There is no any image in your env when you first installed docker, the docker image need be download from the image servers.

# 1.2 download the docker image from docker hub

docker hub: https://hub.docker.com/

in china you could use dao cloud: http://hub.daocloud.io/

this will be fast than docker hub.

you could search any docker images you interested on these two website.

example for ubuntu:

download: http://hub.daocloud.io/repos/4983bbad-a6eb-46c6-abb3-f8e3ff35b2c8

docker pull daocloud.io/library/ubuntu:16.10

# 1.3 configure shared directory of docker container and local host(your windows)

Open the virtualbox and open the virtual machine setting of “default”

![docker5](https://ithorseman.files.wordpress.com/2018/04/docker5.png)

Select the Shared Folders, add the share folder under the item “machine folders”

![docker6](https://ithorseman.files.wordpress.com/2018/04/docker6.png)

![docker7](https://ithorseman.files.wordpress.com/2018/04/docker7.png)

**Attention**: this folder should be your programing woking folder. And you should chose the item “auto-mount” and “make permanent”.

Then, mount the share folder in “default”, open the “default” show

![docker8](https://ithorseman.files.wordpress.com/2018/04/docker8.png)![docker10](https://ithorseman.files.wordpress.com/2018/04/docker10.png)

Execute the command: (if VM configuration is not saved before power off, should execute below command when start VM next)

mkdir /mnt/share

mount -t vboxsf CODING /mnt/share

![docker11](https://ithorseman.files.wordpress.com/2018/04/docker11.png)

# 1.4 pull the docker images and run the docker containers

docker pull daocloud.io/library/ubuntu:16.10

docker create -ti -v //mnt/share:/mnt/share --name=your_container daocloud.io/library/ubuntu:16.10 bash

docker start your_container

docker exec -ti your_container bash

ok, until now, you could use the docker container to do any thing you want on you windows environment.