---
layout: post
title: Docker Swarm tricks
description: "Tricks to setup and use Docker swarm."
tag: Docker
category: DevOps 
date: 2020-01-10 14:13:17
---


## Docker swarm configuration

Set a networkpool for Docker's interface. Defining it in the daemon configuration avoid random 
network that will interfere with any existing network segments on your site.

```json
{
   "insecure-registries":[],
   "live-restore": false,
   "userland-proxy": false,
   "default-address-pools":[
	{"base":"172.100.0.0/16","size":24}
   ]
}
```

## Use Portainer to monitor the swarm('s)

Run [Portainer](https://www.portainer.io/) with this compose file. Portainer gives you a swiss
army knife to monitor and manage your swarm.

```docker
version: '3.4'

services:
  agent:
    image: portainer/agent:1.15.1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer:1.23.0
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
    networks:
      - agent_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]

networks:
  agent_network:
    driver: overlay
    attachable: true

volumes:
  portainer_data:
```

