---
layout: post
title: Monitoring Docker Swarm
description: "Test setup and template for docker monitoring with Prometheus and Grafana."
tag: Docker Grafana Prometheus
category: DevOps 
date: 2020-01-10 14:13:17
---


## Docker Compose

```docker
version: '3.4'

services:

  prometheus:
    image: prom/prometheus:v2.15.2
    ports:
      - "9090:9090"
    networks:
      - app-net

  grafana:
    image: grafana/grafana:6.5.1
    ports:
      - "3000:3000"
    networks:
      - app-net

networks:
  app-net:
```
