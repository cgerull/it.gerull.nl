---
layout: post
title: Shrink logic volumes
description: "Use LVM2 to shrink volumes on the fly."
tags:
  - File System
  - Install
  - Linux
  - Logical Volumes
comment: false
categories:
  - Linux
date: 2014-04-08
---

In this post I will point out the steps to shrink logic volumes.

The order of shrinking is just the reverse of expanding:

* un-mount the file system
* check file system integrity
* shrink file system
* shrink logic volume.

> **Before you start make sure you have recent backup of the disk.** 

## The steps in detail

1\. First un-mount the file system to make sure no one has access anymore. For some file systems you must switch to single user mode for this. Then check the file system integrity.
  - if /yyy is the mount point `sudo umount /yyy`
  - As this is a logical volume, vg-xxx is the volume group and yyy is the symbolic name of the lv. `sudo e2fsck -f /dev/mapper/vg-xxx/yyy`

2\. Shrink the file system to some point
`sudo resize2fs /dev/mapper/vg-xxx/yyy 2000M`
Here it goes down to 2G, but **above** the physical size of existing files. This command works for both ext2 and ext3.

3\. Shrink the logic volume
`sudo lvresize -L -1G /dev/mapper/vg-xxx/yyy`
Here the logic volume is shrunk by 1G. Don't shrink beyond your file system size, the consequences are disastrous. Do the math. _And take it very serious!_

4\. Expand the file system to fit the shrunk logic volume
`sudo resize2fs -p /dev/mapper/vg-xxx/yyy`

5\. Remount the file system.
