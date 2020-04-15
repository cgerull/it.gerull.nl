---
layout: post
title: Vagrant template
description: "Install and run a virtual machine on your host."
tag: Linux Vagrant VirtualBox
category: Linux
date: 2019-09-19 07:13:17
---


Install and run a virtual machine on your host

The template is in public repository in GitHub [Vagrant template](https://github.com/cgerull/vagrant-single)

The installation can be used for local development or testing. Please change
the parameter in Vagrantfile to suit your needs. This script will install 1 virtual
machine with Ansible and Docker provisioned.

You can use the machine to run various docker containers and create a testing / deployment
environment.

## Virtualisation environments
### VirtualBox

Is the default and need nom alterations in the Vagrant file. To install the VBox extentions
you need the vagrant-vbguest plugin. The /etc/hosts sync can be done with vagrant-hosts.

### Hyper-V

When using Hyper-V you need to an elevated command prompt. Run CMD as Administrator Start the
machines with vagrant up --provider=hyperv The /etc/hosts sync can be done with vagrant-hostmanager.

## Operation system

You can change the OS to any supported box you need. In that case please change the setup scripts
the example playbook according to you flavour of Linux.

Default OS is CentOs 7, an entry for Ubunti Bionic can be enabled.
