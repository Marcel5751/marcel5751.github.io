---
layout: post
title: "Connecting Kong Gateway Admin API to Konga"
---

Recently, I was experimenting with different API Gateways/ Management systems, when I ran into a little problem, which took longer than nessessary to figure out.

## Problem
Konga is not able to connect to the Kong Admin API using Key Auth.

![Error shown in Konga Web UI](/public/cant_connect_konga.png)

The Kong admin API connection details can be seeded to Konga like this.

{% highlight js %}
// https://github.com/pantsel/konga/blob/master/docs/SEED_DEFAULT_DATA.md
module.exports = [
    {
        "name": "Kong Test Seed",
        "type": "key_auth",
        "kong_admin_url": "http://host.docker.internal:8001",
        "kong_api_key": "DonKeyKong",
        "health_checks": false,
    }
]
{% endhighlight %}


## Solution

Set the `kong_admin_url` to the Docker host, so `http://host.docker.internal:8001` and `http://kong:8001` (with `kong` beeing the name of the service in docker compose) should work. If it still does not work **Clear Browser Cache or open in private Window**.

With the docker-compose beeing:

```
version: '3'

services:
  webapps:

...

  kong:
    container_name: kong
    image: kong:latest
    volumes:
      - ./kong.yml:/usr/local/kong/declarative/kong.yml
    environment:
      - KONG_DATABASE=off
      - KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yml
      - KONG_PROXY_ACCESS_LOG=/dev/stdout
      - KONG_ADMIN_ACCESS_LOG=/dev/stdout
      - KONG_PROXY_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl
    ports:
      - "8000:8000"
      - "8443:8443"
      - "127.0.0.1:8001:8001"
      - "127.0.0.1:8444:8444"
    networks:
      - kong-net

  
  konga:
    container_name: konga
    image: pantsel/konga
    volumes:
      - ./konga-init:/konga-init
    ports:
      - 1337:1337
    environment:
      - NODE_ENV=development
      - TOKEN_SECRET=wasd
      - KONGA_SEED_USER_DATA_SOURCE_FILE=/konga-init/userdb.data
      - KONGA_SEED_KONG_NODE_DATA_SOURCE_FILE=/konga-init/kong_node.data
    networks:
      - kong-net

networks:
  kong-net:
    driver: bridge

```

And the relevant parts of the kong.yml:

```
_format_version: "2.1"

services:
  - name: admin-api
    url: http://127.0.0.1:8001
    routes:
      - paths:
        - /admin-api
    plugins:
    - name: key-auth

...

consumers:
 - username: admin
   keyauth_credentials:
   - key: DonKeyKong

...

```
