---
layout: post
title: "Connecting Kong Gateway Admin API to Konga"
---

Recently, I was experimnenting with different API Gateways/ Management systems, when I ran into a little problem, which took longer than nessessary to figure out.

## Problem
Konga is not able to connect to the Kong Admin API.

![Error shown in Konga Web UI](/public/cant_connect_konga.png)

The Kong admin API connection details can be seeded to Konga like this 
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

Set the URL to the Docker host.
Clear Browser Cache or open in private Window.