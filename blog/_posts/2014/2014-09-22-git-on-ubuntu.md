---
layout: post
title:  "Git for Ubuntu"
description: "How to install a recent Git version on Ubuntu."
date:   2014-09-22
categories: ubuntu git
tags: git
---

Running a recent Git for Ubuntu can sometimes be problematic. The Git version in the offical Ubuntu archives lags behind the maintainers, stable, version. To solve this install the Maintainers team version.

Add the PPA (personal Packages Archive) to your apt sources

```shell
deb http://ppa.launchpad.net/git-core/ppa/ubuntu precise main

deb-src http://ppa.launchpad.net/git-core/ppa/ubuntu precise main
```
or

```shell
sudo add-apt-repository ppa:git-core/ppa
```

if add-apt-repository is not installed, install it with apt-get install python-software-properties

`sudo apt-get update && sudo apt-get install git`

In a corporate envrionment the case of an unverified key can rise a problem. Cause is that

`apt-key adv --recv-keys ;`

may not work behind some firewalls. to solve this issue, follow these steps:

Open  http://keyserver.ubuntu.com/ in a Browser
and search for the offending key. Don’t forget to mark the string as hex by prepending 0x.
- Get key from Ubuntu keyserver
- Insert into a file
    ```shell
    apt-key add <file>
    apt-get update
    ```

For details see [opensourceforgeeks](http://opensourceforgeeks.blogspot.nl/2013/04/w-gpg-error-httpppalaunchpadnet-precise.html)