---
layout: post
title: Expand logic volumes
description: "Use LVM2 to expand volumes on the fly."
tags:
  - File System
  - Install
  - Linux
  - Logical Volumes
comment: false
categories:
  - Linux
date: 2014-03-23
---

Logical volumes are easy to extend. The basic steps are like this:

1. check free space
2. expand logic volume
3. expand file system.

With lvm2 you can do it even with file system mounted.

Although I recommend to make a backup first and switch to single user mode if it is a server.

Here's the recipe:

1\. Check the mounted file system with

`df -h`

2\. Check the available logic volumes

`sudo lvdisplay`

3\. Check how much free space available in this volume group for the growth

`sudo vgdisplay`

4\. Now resize the logic volume up to some point (of course within the free space limit)

`sudo lvresize -L +1GB /dev/mapper/vg-xxx/yyy`

5\. Check the new size of the logic volume

`sudo lvdisplay`

6\. Expand the existing file system to the size of logic volume

`sudo resize2fs -p /dev/mapper/vg-xxx/yyy`

This command works for both ext2 and ext3 file systems.