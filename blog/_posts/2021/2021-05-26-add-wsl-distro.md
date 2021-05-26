---
layout: post
title: Add distros to WSL
description: "Add unix distros without Microsoft store."
tag: linux wsl vm
category: linux
date: 2021-05-26 21:13:17
---
Microsoft Windows WSL is a very convinient tool to run various virtual Linux machine without much configuration of a hypervisor.

Most Linux distribution can be installed via the Microsoft store. Some are free, but most are charged for.

However it is easy to install an arbitary distribution from a docker container with afew steps.

## Import a Linux rootfs

Prepare a folder for the distributions and create a rootfs tar.

```powershell
mkdir images
mkdir wsl

docker run --name centos7 centos:7
docker export -o centos-7-rootfs.tar centos7
```

Now import the rootfs archive into WSL.

```powershell
wsl --import centos7 .\wsl\centos7 .\images\centos-7-rootfs.tar
```

## Run the Linux vm

Start the distro.

```powershell
wsl -d centos7
```
