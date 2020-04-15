---
layout: post
title: Install a recent Git on CentOS 7
description: "Two ways to install a recent Git version on CentOS."
tag: Git CentOS
category: Git
date: 2019-08-05 08:13:17
---

CentOS 7 as well as RedHat 7 has very old Git version (1.8.x) in the official repositories. The
most tools I use need at least Git > 2.0. 
In this post I will describe in brief how to update your Git installation. Here I will describe just two 
of the possbile solutions. Install from source and install from the [IUS](https://ius.io/) repositories.

## Preperation

To avoid entangeling 2 versions of Git I first deinstall the official package.

```
sudo yum remove -y git*
```

## Install via IUS

Now install the IUS community repository. This command add the repository in the same way the epel repository is installed.

```
sudo yum install -y  https://centos7.iuscommunity.org/ius-release.rpm
```

Now I can install the Git suite.

```
sudo yum install -y  git2u-all
```

That's it, keep an eye for security announcement on [Git](https://git-scm.com/) and on the [IUS repository](https://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/repoview/git2u-all.html)

## Install from source

If the development tools are not already installed I need to install them first. For a vanilla CentOS these packages are needed.

```
sudo yum install -y wget perl-CPAN gettext-devel perl-devel  openssl-devel  zlib-devel https-devel
```

Next get the Git source tree, build and install. In the example I use Git version 2.22.0

```
wget https://github.com/git/git/archive/v2.22.0.tar.gz
tar -zxf v2.22.0.tar.gz
cd git-2.22.0
sudo make install
```

Please in mind that I'm now my own maintainer, so i need to closly watch the Git annoouncement for security and bug fixes.




