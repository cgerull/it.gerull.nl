---
layout: post
title: Install and configure automaount on CentOS 7
description: "Notes on autofs on RH family Linux."
tag: Centos Linux
category: Linux
date: 2019-07-12 10:13:17
---

## Prerequisities

```bash
# First install autofs
yum install -y autofs cifs-utils

# Now enable and , if ready, start the daemon
systemctl enable autofs
```

## Configuration

The system is now ready for configuration.

### autofs system configuration

Normally `/etc/sysconfig/autofs` is ready to use. However to get more debug logging uncomment add these options.

```bash
# Use OPTIONS to add automount(8) command line options that
# will be used when the daemon is started.
#
OPTIONS="--debug --verbose"
#
```

### Main configuration is in `/etc/auto.master`

```bash
#
# This is a 'master' automounter map and it has the following format:
# mount-point [map-type[,format]:]map [options]
# For details of the format look at auto.master(5).
#
/misc   /etc/auto.misc
/vagrant    /etc/auto.vagrant
#
# NOTE: mounts done from a hosts map will be mounted with the
#       "nosuid" and "nodev" options unless the "suid" and "dev"
#       options are explicitly given.
#
/net    -hosts
#
# Include /etc/auto.master.d/*.autofs
# The included files must conform to the format of this file.
#
+dir:/etc/auto.master.d
#
# Include central master map if it can be found using
# nsswitch sources.
#
# Note that if there are entries for /net or /misc (as
# above) in the included master map any keys that are the
# same will not be seen as the first read key seen takes
# precedence.
#
+auto.master
```

### Configuration for a SMB share

To avoid hardcoding your credentials into world readable file create a credential file
in your home.

```bash
# User credentials for SMB share
username=<username>
password=<password>
# Optional
domain=<domain name>
```

```bash
#
# This is an automounter map and it has the following format
# key [ -mount-options-separated-by-comma ] location
# Details may be found in the autofs(5) manpage

# folder to mount = `provisioning`
# mount options = `-fstype=cifs,credentials=/home/vagrant/.secret`
# share URL = `://NLDT-11695/provisioning`
provisioning   -fstype=cifs,credentials=/home/vagrant/.secret    ://NLDT-11695/provisioning
```

## Running

Start the autofs daemon with `systemctl start autofs`

Try `ls <mounted directory>`, e.g `ls /vagrant/provisioning` the automounter should now mount the share.

If anything goes wrong, check `/var/log/messages`

> Once you configuration is running, you can uncomment the verbose logging options in
`/etc/sysconfig/autofs`.

See also:

- [RedHat mounting ans SMB share](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/mounting_an_smb_share)
- [How to configure autofs](https://linuxconfig.org/how-to-configure-the-autofs-daemon-on-centos-7-rhel-7)
