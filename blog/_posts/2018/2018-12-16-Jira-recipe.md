---
layout: post
title: Jira recipe
description: "Install or upgrade Atlassian Jira."
tag: Jira Atlassian
category: Atlassian
date: 2018-12-16 017:37:10
---
# Jira recipe

This instruction assumes that Jira is installed in `/opt/jira` and the data directory is `/var/jira`.

## Prepration

Check if the requirements in the supported platforms are met.

Please read the release notes, adjust the version number in the URL.

Download binary Jira installer from Atlassian (replace the JIRA version with the required one). Go to the [Jira download page](https://www.atlassian.com/software/jira/download) to find the correct version.

```bash
cd /opt/download
curl -vvkL https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-M.m.p-x64.bin > atlassian-jira-software-M.m.p-x64.bin
# M = major version
# m = minor version
# p = patch level
# e.g. 7.5.3
chmod u+x atlassian-jira-software-M.m.p-x64.bin
```

>/opt needs at least 800M of free space, clean-up or extend filesystem

### Perform these steps

- If on a virtual host, take a snapshot of VM
- If upgrading, check if last backup was succesfull ls -l /var/pgbackup
- In the Add-ons section upgrade all plugins
- Then do the Jira upgrade test
- Perform a reindex
- Run an integrity check

## Upgrade / Install

```bash
cd /opt
./download/atlassian-jira-software-M.m.p-x64.bin
```

- Press o for OK
- if new, install choose 2 for Custom Installl; if upgrade, choose 3  Upgrade an existing JIRA installation
- installation directory is `/opt/jira`
- if upgrade, answer n to 'Back up JIRA home diretory'
- if upgrade and modified files were found, copy modified files to `/var/tmp`
- if upgrade, press u; if new install press i
- choose n on Start JIRA Software M.m.p now?

If JIRA is started at the end of the installation, please shutdown via `/opt/jira/bin/shutdown.sh`.

If installation is not in the desired location move installation from `/opt/atlassian/jira/` to `/opt/jira`.

## Postinstall

- check if the file and directory permission are correct
- check `/etc/init/jira.conf` or `/etc/init.d/jira` to ensure JIRA is run as user JIRA
- check the settings in `/opt/jira/bin/setenv.sh` (check with saved file from previous version). For SDMC add the required JAVA Arguments.
- edit `/opt/jira/conf/server.xml` enable AJP (Uncomment the AJP connector on port 8009)
- only if exists (JIRA 5 / 6) edit `/opt/jira/bin/permgen.sh`   (No longer applicable , we can Skip this step)
- disable password autocompletion
- start JIRA and monitor the log file with tail -f `/var/jira/log/atlassian-jira.log`

## Finalizing

- check Add-ons for possible updates and execute the updates.
- perform a re-index. (System | Advanced)
- do a integrity check. (System | Troubleshooting and support)
- run the support tool and check any issues. (System | Troubleshooting and support)
- if new install, add the machine to your backup routine.
