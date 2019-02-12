---
layout: post
title: Reorganize physical volumes
description: "A guide to reorganize Linux LVM disks."
tag: linux lvm partitions
category: Linux
date: 2019-01-10 08:13:17
---


These description is written for Ubuntu Linux. With a bit of creativity it should also work for other Linux flavours.


> For all resize operations it is mandatory to stop running applications. Most reduce operations expect unmounted filesystems too.

## Free or rearrange physical volumes
Suppose you have a cluttered (physical) disklayout, i.e a number of small disk added over the course of time to extend the logical volume. Sometimes it's convenient to consolidated the small (virtual-) disk into a large one.

The general idea is to add one temporary large virtual disk that can contain all other volumes, move the the physical volumes to the temporary volume. Then delete the small partition and create a sufficiently large one. Eventually reserve extra space for expansion in the near future. Once the volume is created move the physical volume from temporary to the newly created one.

1. Make sure you have a volume with enough free space to contain the emptied volume(s), i.e. add a temporary disk to the VM.
1. pvmove <source device> <destination temporary device>
1. vgreduce <volumegroup> <physical device>
1. pvremove <physical device>
1. Consolidate the old partitions and create one LVM partion
1. pvcreate <physical device from step 5>
1. vgextend <volumegroup> <physical device>
1. pvmove <source temporary device> <destination device>
1. vgreduce <volumegroup> <temporary device>
1. pvremove <temporary device>
1. Remove temporay volume from VM.


As the steps above are very elementary, here is an step-by-step example of a space consolidation. 

## Resize physical partition

1. Stop applications
1. umount filesystem(s)
1. pvresize --setphysicalvolumesize <newsize>GB /dev/<physical volume>
1. parted → resizepart → <partnumber> → End

