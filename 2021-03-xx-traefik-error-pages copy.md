---
title: 'Using #traefik error pages to handle unavailable services'
date: 2021-03-xxT16:02:00+01:00
author: Nicholas Dille
layout: post
permalink: /blog/2021/03/xx/using-traefik-error-pages-to-handle-unavailable-services/
categories:
  - Haufe-Lexware
tags:
- Container
- Docker
- traefik
---
XXX

<img src="/media/2021/03/error-2129569_1920.jpg" style="object-fit: cover; object-position: bottom; width: 100%; height: 250px;" />

<!--more-->

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

## Catching unavailable services

XXX

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

  error-pages:
    image: nginx:latest
    volumes:
    - ./error-pages/pages:/usr/share/nginx/error-pages
    - ./error-pages/default.conf:/etc/nginx/conf.d/default.conf
    labels:
      traefik.enable: "true"
      traefik.http.services.error-pages-service.loadbalancer.server.port: 80
      traefik.http.routers.error-pages.entrypoints: http
      traefik.http.routers.error-pages.rule: HostRegexp(`{host:.+}`)
      traefik.http.routers.error-pages.priority: 1
      traefik.http.routers.error-pages.middlewares: error-pages-middleware
      traefik.http.middlewares.error-pages-middleware.errors.status: 404
      traefik.http.middlewares.error-pages-middleware.errors.service: error-pages-service
      traefik.http.middlewares.error-pages-middleware.errors.query: /404.html

  www:
    image: nginx:stable
    volumes:
    - ./service:/usr/share/nginx/html
    labels:
      traefik.enable: "true"
      traefik.http.services.www.loadbalancer.server.port: 80
      traefik.http.routers.www.entrypoints: http
      traefik.http.routers.www.rule: HostRegexp(`www.localhost`)
      traefik.http.routers.www.priority: 100
```

XXX

```bash
curl -sv --resolve www2.localhost:80:127.0.0.1 http://www2.localhost
```

XXX

```bash
docker-compose config \
    --file docker-compose.yaml \
    --file docker-compose.catch-all.yaml
```

## Returning custom error pages

XXX

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

  error-pages:
    image: nginx:latest
    volumes:
    - ./error-pages/pages:/usr/share/nginx/error-pages
    - ./error-pages/default.conf:/etc/nginx/conf.d/default.conf
    labels:
      traefik.enable: "true"
      traefik.http.services.error-pages-service.loadbalancer.server.port: 80
      traefik.http.routers.error-pages.entrypoints: http
      traefik.http.routers.error-pages.rule: HostRegexp(`{host:.+}`)
      traefik.http.routers.error-pages.priority: 1
      traefik.http.routers.error-pages.middlewares: error-pages-middleware
      traefik.http.middlewares.error-pages-middleware.errors.status: 404
      traefik.http.middlewares.error-pages-middleware.errors.service: error-pages-service
      traefik.http.middlewares.error-pages-middleware.errors.query: /404.html
      traefik.http.middlewares.outage-middleware.errors.status: 500-599
      traefik.http.middlewares.outage-middleware.errors.service: error-pages-service
      traefik.http.middlewares.outage-middleware.errors.query: /5xx.html

  www:
    image: nginx:stable
    volumes:
    - ./service:/usr/share/nginx/html
    labels:
      traefik.enable: "true"
      traefik.http.services.www.loadbalancer.server.port: 80
      traefik.http.routers.www.entrypoints: http
      traefik.http.routers.www.rule: HostRegexp(`www.localhost`)
      traefik.http.routers.www.priority: 100
      traefik.http.routers.www.middlewares: outage-middleware

  httpbin:
    image: kennethreitz/httpbin
    labels:
      traefik.enable: "true"
      traefik.http.services.bin.loadbalancer.server.port: 80
      traefik.http.routers.bin.entrypoints: http
      traefik.http.routers.bin.rule: HostRegexp(`bin.localhost`)
      traefik.http.routers.bin.priority: 100
      traefik.http.routers.bin.middlewares: outage-middleware
```

XXX

```bash
# TEST
```

XXX

```bash
docker-compose config \
    --file docker-compose.yaml \
    --file docker-compose.catch-all.yaml \
    --file docker-compose.internal-error.yaml
```

## More to come

XXX see part 2