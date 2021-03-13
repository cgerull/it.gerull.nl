---
layout: post
title: Build a k3s cluster in Raspberry Pi.
description: "Configure a lightweight k3s cluster on Arm hardware."
tag: kubernetes k3s raspberrypi
category: Kubernetes
date: 2021-01-16 15:13:17
---
Testing your containers and Kubernetes with Minikube on your local machine is fine. But if you have 2 more Raspberry Pi’s (from V.2 upwards) it’s more fun and gives you the real ‘cluster feeling’. It also gives you the opportunity to test your container images on ARM processors.

This post is a description of my pi cluster installation. In my case I use 3 Raspberries: a Pi 4B with 4GB and a 128GB thumb drive, a Pi3bplus with 2GB and a Pi2b with 2GB.

## Requirements

The pi’s should be installed, configured on your local network and you should be able to access them via ssh from your workstation.

Set up your kernel option during system boot.

```bash
# Add cgroup configuration to cmdline.txt
# On rasperige system it's in /boot/
# On ubuntu 20.04 system's go to /boot/firmware
#
echo "cgroup_enable=cpuset \
cgroup_enable=memory \
cgroup_memory=1" >> /boot/firmware/cmdline.txt
```

## Install k3s

Login in to your designated master.

```bash
# Download and install k3s
curl -sfL https://get.k3s.io | sudo sh -
# Check if the server is running
sudo systemctl status k3s
sudo k3s kubectl get nodes
```

Now it’s time to install the worker nodes.

```bash
# Get the node token on the master
sudo cat /var/lib/rancher/k3s/server/node-token
#
# Now go the workers and install the agent
# Change URL and TOKEN to your values
sudo su -
curl -sfL https://get.k3s.io \
| K3S_URL="https://myserver:6443" \
K3S_TOKEN=xxxx \
sh -
#
# Check the installation on the master
sudo k3s kubectl get nodes
```

## Configure

To access the cluster from our workstation we need to install the credentials.

```bash
# The quick way when you have a user with sudo rights
# and you don't have active configuration.
# Assuming your user is myuser and your k3s master is
# k3s-master
ssh myuser@k3s-master sudo cat /etc/rancher/k3s/k3s.yaml >> .kube/config
#
# The verbose commands are
# copy the credentials from the master
sudo cat /etc/rancher/k3s/k3s.yaml
# Paste them on your workstation into $HOME/.kube/config
```

The config file should look like this.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS...g==
    server: https://10.103.9.189:6443
  name: mycluster
contexts:
- context:
    cluster: mycluster
    namespace: test
    user: kubernetes-admin
  name: mycluster
current-context: mycluster
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS...Qo=
    client-key-data: LS...Cg==
```

## Uninstall

To remove the Kubernetes cluster start uninstalling the agent from workers and finally remove the master.

```bash
# On the agents
/usr/local/bin/k3s-agent-uninstall.sh
#
# On the server
/usr/local/bin/k3s-uninstall.sh
```