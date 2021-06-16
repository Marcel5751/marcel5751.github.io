---
layout: post
title: "Fun with MongoDB: Referencing of Documents with Spring Data Mongo"
---

Recently, I was experminenting with different API Gateways/ Management systems. 

## Problem

Konga is not able to connect to the Kong Admin API.

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
Refresh Browser or o