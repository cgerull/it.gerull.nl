---
layout: post
title: Confluence recipe
description: "Steps to install / upgrade Confluence."
tag: Confluence Atlassian
category: Atlassian
date: 2018-12-18 08:13:17
---
# Confluence recipe

This instructions assume that Confluence is installed in `/opt/confluence` and the data directory is `/var/confluence`.

## Preparation

- [ ] upgrade / install database
- [ ] remove stale installers and backups
- [ ] check diskspace, /opt needs at least 900MB free space
- [ ] check if the license is valid
- [ ] download Confluence


## Confluence download

```
#
# replace <version> with the required version, e.g. 6.4.3
#
cd /opt/download
sudo curl -vvkL https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-<version>-x64.bin > atlassian-confluence-<version>-x64.bin
sudo chmod +x atlassian-confluence-<version>-x64.bin
```

## Upgrade

- check local admin account, reset password if necessary
- stop the server
- execute `/opt/downloads/atlassian-confluence-5.9.6-x64.bin`
- if behind a proxy then add proxy configuration to `/opt/confluence/bin/setenv.sh`

```
CATALINA_OPTS="-Dhttp.proxyHost=<http-proxy> -Dhttp.proxyPort=8080 ${CATALINA_OPTS}"
CATALINA_OPTS="-Dhttps.proxyHost=<https-proxy> -Dhttps.proxyPort=8080 ${CATALINA_OPTS}"
CATALINA_OPTS="-Dhttp.nonProxyHosts=localhost\|127.0.0.1\|*.<domain>\|*.<domain>\|*.<domain> ${CATALINA_OPTS}"
```
- When behind Apache with mod_jk, add AJP to `/opt/confluence/conf/server.xml` or uncomment if avaiable
- Set permissons to `<confluenceuser>.<confluencegroup>` for `/opt/confluence` and `/var/confluence` and their subdirectories
- Start the server
- View the logfile and check upgrade / startup errors (`tail -f /var/confluence/logs/atlassian-confluence.log`)
- Update plugins
- Run support tools -> instance health check

## Important

Add the AJS connector only after the default http connector (port 8080 or 8090) otherwise the syncrony process will break.

```
<Connector port="8009" enableLookups="false" redirectPort="8443" protocol="AJP/1.3" URIEncoding="UTF-8"/>
```

## Crowd connection

If the connection with crowd is broken you will see connection errors in the log

To re-configure the connection follow these steps:

1) login as local admin
1) test, and if necessary reconfigure, crowd
1) sync crowd
1) login as LDAP user


