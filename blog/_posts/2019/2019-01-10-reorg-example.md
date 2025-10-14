---
layout: page
title: "LVM reorganisation"
description: "Example LVM reorganisation of a Linux machine. Moving several partions into one."
tag: lvm linux
category: Linux
date: 2019-02-01 20:44:17
---

## Before

### pvdisplay

```bash
sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               vg0
  PV Size               7.76 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1986
  Free PE               0
  Allocated PE          1986
  PV UUID               77oIcj-anLd-Beo6-CHB3-1OvS-7cnR-QAYR9g

  --- Physical volume ---
  PV Name               /dev/sda6
  VG Name               vg0
  PV Size               24.00 GiB / not usable 992.50 KiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              6144
  Free PE               0
  Allocated PE          6144
  PV UUID               XR3z9J-2Iet-rv2G-bxpf-ICY6-AuVp-pfv0OH

  --- Physical volume ---
  PV Name               /dev/sda7
  VG Name               vg0
  PV Size               8.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2047
  Free PE               0
  Allocated PE          2047
  PV UUID               a861DI-fmlH-PGTH-FTCo-wQku-O8PS-dI93Gb

  --- Physical volume ---
  PV Name               /dev/sda8
  VG Name               vg0
  PV Size               10.00 GiB / not usable 3.97 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               0
  Allocated PE          2559
  PV UUID               o3rier-6DtA-1nje-0SBl-LKmv-RmRb-q3LFmA

  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               vg0
  PV Size               50.00 GiB / not usable 3.31 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              12799
  Free PE               538
  Allocated PE          12261
  PV UUID               T9s72k-7tgH-QROi-wKNx-gbn8-dgQy-BqHi6e
```

### vgs

```bash
sudo vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg0    5  10   0 wz--n- 99.75g 2.10g
```

### fdisk

```bash
sudo fdisk -l /dev/sda

Disk /dev/sda: 53.7 GB, 53687091200 bytes
255 heads, 63 sectors/track, 6527 cylinders, total 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00073bcb

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      499711      248832   83  Linux
/dev/sda2          501758   104857599    52177921    5  Extended
/dev/sda5          501760    16775167     8136704   8e  Linux LVM
/dev/sda6        16775231    67108863    25166816+  8e  Linux LVM
/dev/sda7        67110912    83886079     8387584   8e  Linux LVM
/dev/sda8        83886143   104857599    10485728+  8e  Linux LVM

sudo fdisk -l /dev/sdb

Disk /dev/sdb: 53.7 GB, 53687091200 bytes
255 heads, 63 sectors/track, 6527 cylinders, total 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x394cc80b

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1              63   104856254    52428096   8e  Linux LVM
```

## Consolidation

Start by cleaning the old snapshots.

We see that LVM uses almost 100GB. It make sense to add an additional virtual disk to the VM as temporary space. After notifing the kernel we create 1 partition of type Linux VM.

Create a new snapshot.

### Cleanup old physical volumes

#### Add temporary drive

```bash
sudo ls /sys/class/scsi_host/  # get the scsi host id, the new drive is probably the latest
sudo $ echo "- - -" > /sys/class/scsi_host/<host#>/scan
sudo fdisk /dev/sdc # create a Linux LVM partition interactivly
```

#### Add temp disk to LVM

```bash
sudo pvcreate /dev/sdc1
sudo vgextend vg0 /dev/sdc1
```

The next is to move all data from the existing physical volumes to the temporary space. This can take more than 1 hour.

## Move physical volumes to temporary space

```bash
sudo pvmove /dev/sdb1 /dev/sdc2
sudo pvmove /dev/sda8 /dev/sdc2
sudo pvmove /dev/sda7 /dev/sdc2
sudo pvmove /dev/sda6 /dev/sdc2
sudo pvmove /dev/sda5 /dev/sdc2
```

Now all data is on the temporary space we can verify and remove the old phy. volumes.

## Verify and remove old volumes

```bash
sudo vgdisplay # should have at least 100GB free
sudo pvdisplay # /dev/sdc1 (and sda8, sda7, sda6, sda5) should have 100% free space
sudo vgreduce vg0 /dev/sdb1
sudo vgreduce vg0 /dev/sda8
sudo vgreduce vg0 /dev/sda7
sudo vgreduce vg0 /dev/sda6
sudo vgreduce vg0 /dev/sda5

sudo pvdisplay  # should show the freed volumes as 'new'
sudo pvremove /dev/sdb1
sudo pvremove /dev/sda8
sudo pvremove /dev/sda7
sudo pvremove /dev/sda6
sudo pvremove /dev/sda5
```

Test if the application is working. As we only shifted blocks of data, the operation of OS and application should be unaltered.

## Consolidate partitions

The old volumes are removed. Now we can remove the old partions with fdisk.

### fdisk remove partions

```bash
sudo fdisk /dev/sdb # remove all partitions
sudo fdisk /dev/sda # remove all Linux LVM partitions, LEAVE sda1 AS IT IS
```

Remove the snapshot.

Switch to VCenter and remove the second virtual disk (sdb). Now extend the first virtual disk 120 GB (the 100GB that are currently in use and 20% for better performance)

Create another snapshot.

Now create one extented partion with 1 logical partion, first notify the kernel about the changed disk size.

### fdisk create new logical partition

```bash
sudo ls /sys/class/scsi_device/     # get the id of your first scsi disk
sudo echo 1 >  /sys/class/scsi_device/X:X:X:X/device/rescan
sudo fdisk /dev/sda # interactivly create a logical partition sda5 with 120GB size and type Linux LVM.
```

## Move data to the new volume

### Add new disk to LVM

```bash
sudo pvcreate /dev/sda5
sudo vgextend vg0 /dev/sda5
```

The next is to move the data back from temporary space to the newly created volume. This can take more than 1 hour.

### Move physical volumes to destination space

```bash
sudo pvmove /dev/sdc2 /dev/sda5
```

### Verify and remove temporary space

```bash
sudo vgdisplay # should have at least 120GB free
sudo pvdisplay # /dev/sdc1 should have 100% free space
sudo vgreduce vg0 /dev/sdc1
sudo pvdisplay  # should show the freed volumes as 'new'
sudo pvremove /dev/sdc1
```

Test if the application is working. As we only shifted blocks of data, the operation of OS and application should be unaltered.

## Finalizing

Remove the temporary drive from VCenter.

## After

The VM has now 1 logical partion for LVM and at moment 1 physical device. This gives us more space in the future to add ad-hoc devices again.
