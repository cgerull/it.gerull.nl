---
layout: post
title: HA Proxy as a reverse ssh proxy
description: "Configure haproxy to act as reverse proxy for ssh."
tags:
  - Proxy
  - SSH
  - Linux
comment: false
categories:
  - Linux
date: 2015-08-23
---

If you want to hide a ssh service to the public use a reverse ssh proxy. Just 
as Apache can act as a reverse http / https proxy, so can HA Proxy act as a 
reverse ssh proxy.

Why just don’t use port forwarding? Well, suppose you have a service like 
GitHub,GitLab or Bitbucket to serve your upstream Git repository and you want to access your 
reverse proxy host and your service host by ssh on port 22? Then [HAProxy](http://www.haproxy.org) is in my 
opinion the easiest option to forward the SSL service port to a backend server.

Bear in mind to upgrade your firewall on the proxy host.

Here is a sample configuration file.

```shell
# This config needs haproxy-1.1.28 or haproxy-1.2.1 
# Customized to use a ssh proxy

global
log 127.0.0.1 local0
#log 127.0.0.1 local1 notice
#log loghost local0 info
maxconn 4096
user haproxy
group haproxy
daemon
#debug
#quiet

defaults
log global
option dontlognull
retries 3
option redispatch
maxconn 2000
contimeout 5000
clitimeout 50000
srvtimeout 50000

frontend sshd
bind *: <port>

default_backend ssh
timeout client 1h

backend ssh
mode tcp
server <server-ip:check port
```
