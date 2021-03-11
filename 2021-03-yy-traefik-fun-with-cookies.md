---
title: 'Using #traefik to display maintenance information'
date: 2021-03-yyT16:02:00+01:00
author: Nicholas Dille
layout: post
permalink: /blog/2021/03/yy/using-traefik-to-display-maintenance-information/
categories:
  - Haufe-Lexware
tags:
- Container
- Docker
- traefik
---
XXX

# https://unsplash.com/photos/kID9sxbJ3BQ
<img src="/media/2021/03/error-2129569_1920.jpg" style="object-fit: cover; object-position: bottom; width: 100%; height: 250px;" />

<!--more-->

XXX reference "part 1"

## traefik and the service

XXX

XXX https://github.com/nicholasdille/traefik-maintenance

```yaml
version: "3.7"

services:

  traefik:
    image: traefik:v2.4
    command:
    - --log=true
    - --log.level=DEBUG
    - --api.dashboard=true
    - --entrypoints.http.address=:80
    - --providers.docker=true
    - --providers.docker.exposedByDefault=false
    ports:
    - 127.0.0.1:80:80
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /etc/localtime:/etc/localtime:ro
    restart: always
    labels:
      traefik.enable: "true"
      traefik.http.routers.traefik.entrypoints: http
      traefik.http.routers.traefik.service: api@internal
      traefik.http.routers.traefik.rule: HostRegexp(`traefik.localhost`)

  www:
    image: nginx:stable
    volumes:
    - ./service:/usr/share/nginx/html
    labels:
      traefik.enable: "true"
      traefik.http.services.www.loadbalancer.server.port: 80
      traefik.http.routers.www.entrypoints: http
      traefik.http.routers.www.rule: HostRegexp(`www.localhost`)
```

XXX add to `hosts` or run:

```bash
curl -sv --resolve www.localhost:80:127.0.0.1 http://www.localhost
```

XXX

```bash
docker-compose config \
    --file docker-compose.yaml
```

## XXX

XXX maintenance

```yaml
version: "3.7"

services:

  traefik:
    image: traefik:v2.4
    command:
    - --log=true
    - --log.level=DEBUG
    - --api.dashboard=true
    - --entrypoints.http.address=:80
    - --providers.docker=true
    - --providers.docker.exposedByDefault=false
    ports:
    - 127.0.0.1:80:80
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /etc/localtime:/etc/localtime:ro
    restart: always
    labels:
      traefik.enable: "true"
      traefik.http.routers.traefik.entrypoints: http
      traefik.http.routers.traefik.service: api@internal
      traefik.http.routers.traefik.rule: HostRegexp(`traefik.localhost`)

  www:
    image: nginx:stable
    volumes:
    - ./service:/usr/share/nginx/html
    labels:
      traefik.enable: "true"
      traefik.http.services.www.loadbalancer.server.port: 80
      traefik.http.routers.www.entrypoints: http
      traefik.http.routers.www.rule: HostRegexp(`www.localhost`) && HeadersRegexp(`Cookie`, `maintenance-override=true`)
```

XXX

```bash
docker-compose config \
    --file docker-compose.yaml \
    --file docker-compose.maintenance.yaml
```

## XXX

XXX motd

```yaml
version: "3.7"

services:

  traefik:
    image: traefik:v2.4
    command:
    - --log=true
    - --log.level=DEBUG
    - --api.dashboard=true
    - --entrypoints.http.address=:80
    - --providers.docker=true
    - --providers.docker.exposedByDefault=false
    ports:
    - 127.0.0.1:80:80
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /etc/localtime:/etc/localtime:ro
    restart: always
    labels:
      traefik.enable: "true"
      traefik.http.routers.traefik.entrypoints: http
      traefik.http.routers.traefik.service: api@internal
      traefik.http.routers.traefik.rule: HostRegexp(`traefik.localhost`)

  www-motd:
    image: nginx:stable
    volumes:
    - ./service-motd/pages:/usr/share/nginx/html
    - ./service-motd/default.conf:/etc/nginx/conf.d/default.conf
    labels:
      traefik.enable: "true"
      traefik.http.services.www-motd.loadbalancer.server.port: 80
      traefik.http.routers.www-motd.entrypoints: http
      traefik.http.routers.www-motd.rule: HostRegexp(`www.localhost`)
      traefik.http.routers.www-motd.priority: 90

  www:
    image: nginx:stable
    volumes:
    - ./service:/usr/share/nginx/html
    labels:
      traefik.enable: "true"
      traefik.http.services.www.loadbalancer.server.port: 80
      traefik.http.routers.www.entrypoints: http
      traefik.http.routers.www.rule: HostRegexp(`www.localhost`) && HeadersRegexp(`Cookie`, `motd-read=true`)
```

XXX

```bash
docker-compose config \
    --file docker-compose.yaml \
    --file docker-compose.motd.yaml
```
