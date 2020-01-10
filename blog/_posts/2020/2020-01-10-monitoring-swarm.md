

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
