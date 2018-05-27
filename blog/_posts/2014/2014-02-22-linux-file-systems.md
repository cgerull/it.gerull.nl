---
layout: post
title: Linux File System
description: My implementation of Linux FHS
tags:
  - File System
  - Install
  - Linux
comment: false
categories:
  - Linux
date: 2014-02-22
---

This is just a short example how I use the Linux file system. A full discussion with lot's of links you will find on [Wikipedia](http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard "Wikipedia"). The home of the Linux File System Hierarchy is [here](http://www.pathname.com/fhs/ "FHS").
The layout below is the (almost) default of Debian and Ubuntu distributions.

**/home**

Just for personal settings, annotations and workspaces.

**/tmp**

Classic place to put everything short living. Probably best to (automatically) clean it out regularly.

**/var/www**

Static and dynamic websites

**/var/database**

Database data-files, if big put on its own logical volume.

**/var/&lt;app-name&gt;**

Any other non-web, non-db data or archive files.


Other distributions use slightly different file system layouts and the data-files are maybe in different folders, but as mentioned earlier this is my personal approach.