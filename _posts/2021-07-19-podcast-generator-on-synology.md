---
layout: post
title: "Not able to update windows because virtualbox is installed"
---

Recently, I wanted to put some audio files into a RSS feed.

## Basic setup

On the synology nas, there is the `web` folder which is supposed to contain the files (html, php, css, etc.) needed for a static website.
That means we need to extract the files  inside the zip to new folder e.g. `/volume1/web/podcast-generator`. 
By creating a `virtual Host` and referencing that folder, the Podcast generator should become reachable at podcasts.example.com.

So far so good, but there are a couple of thumbling blocks on the way.

## 1. gettext php Extention is missing
With the default php settings, the website does not display correctly. 
by checking the nginx logs at, we find out that it 
After quick googleing it turs out that this error is due to the missing gettext extention.

In order to activate the gettext extention we need to go back to the Web Station inside DSM and change the PHP Settings.



## 1. Maximum size of files

By default, the size of files that can be uploaded is limited to 32MB. Since podcasts can often be a larger 