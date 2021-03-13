---
layout: post
title: Prepare your workstation as Kubernetes client
description: "Configure and install Kubernetes tools."
tag: kubernetes kubectl helm
category: Kubernetes
date: 2021-01-17 21:13:17
---
To configure, install services and troubleshoot a Kubernetes cluster you need a couple programs and a configuration with credentials.

This post is a short description how I install kubectl and helm. The installation should work on a Mac and on Linux (WSL as well). You find the full description of kubectl on the Kubernetes site. There is are also the commands to install the client on Windows.

## kubectl installation

First the easy way with package managers.

```bash
# Install with snap on Ubuntu
snap install kubectl --classic

# Install with home-brew on Linux or Mac
brew install kubectl

# With apt on Debian family Linux
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Next the installation of the kubectl go binary with curl.

```bash
# Install the latest stable version.
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

# To install a particular version use
curl -LO https://storage.googleapis.com/kubernetes-release/release/<version>/bin/linux/amd64/kubectl

# Now make the binary executable and move to a directory
# in your path
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

## Configure kubectl

Get the configuration keys from your target Kubernetes cluster. Here is an example for a k3s cluster.

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

Your config should look like this.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <cluster ca certificate>
    server: https://<my-cluster-ip:6443
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
    client-certificate-data: <client certificate>
    client-key-data: <client certificate key>
```

## Install helm

Here it goes like the kubectl install. Easiest via a package manager.

```bash
# snap on Ubuntu
snap install helm --classic

# home-brew on Mac or Linux
brew install helm

# chocolatey on Windows
choco install kubernetes-helm
```

Or you can use the installer script.

```bash
# Get the script from GitHub
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

# Run the installer
chmod 700 get-helm.sh
./get-helm.sh
```

That’s it, helm do not need a configuration as it uses the kubectl configuration.

