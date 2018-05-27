---
layout: post
title: Volume layout
description: Linux FHS with LVM2
tags:
  - File System
  - Installation
  - Linux
  - Logical Volumes
comment: false
categories:
  - Linux
date: 2014-03-01
---

Here's my personal preference for an allround Linux box. The following volume 
layout is a baseline. If the machine is a server reduce /home and add much more 
to /var where normally you data lives. For a development box increase /home 
for all your projects.

I use the lvm2 package, so these commands use the most common options.

## First harddrive

### Prepare the disk with fdisk /dev/sda

1. Use fdisk /dev/sda to partion your first drive
  * /dev/sda1 > /boot ext3, 256MB
	* /dev/sda2 > Linux LVM (8e), rest of the drive

### Chop LVM partion into volumes with lvm2 tools

1. pvcreate /dev/sda2  # create the physical volume
2. vgcreate vg0 /dev/sda # create one volume group for the system volumes
3. lvcreate -L 2G - n root vg0   # format as ext3 and mount on /
4. lvcreate -L &lt;depending on memory and disksize&gt; -n swap vg0 # use as swaparea
5. lvcreate -L 2G -n usr vg0   # format as  ext3 mount on /usr
6. lvcreate -L 512M -n tmp -vg0   # format as  ext3, mount on /tmp
7. lvcreate -L &lt;512M - 2G&gt; -n home vg0   # format as  ext3, mount on /home
8. lvcreate -L 256M -n opt vg0   # format as  ext3, mount on /opt
9. lvcreate -L 512M -n usrlocal vg0   # format as  ext3, mount on /usrlocal
10. lvcreate -L &lt;1G - nG&gt; -n var vg0   # format as  ext3, mount on /var

## Additional harddisks

When you install / assign more than one drive you can must create a LVM partiton with fdisk and then you can extend the physical volume. It's up to you to extend the volume group too or create a second vg.
In case of extenting the first vg you can extent the logical volume where the space is needed. If you have installed an application you can also dedicate a logical volume to that application and mount it where approbiate.

