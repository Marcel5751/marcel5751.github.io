---
layout: post
title: "Host Podcast Generator on Synology NAS"
tags: Synology
---

Recently, I wanted to put some audio files into an RSS feed that is served via the Synology NAS.
After some research, I found the Podcast Generator Project [on Github](https://github.com/PodcastGenerator/PodcastGenerator) which makes it very easy to manage an RSS podcast feed.

 <!--more-->

## Basic setup

On the Synology Nas, we can use the Webstation to host the podcast generator code with Nginx and PHP by placing the files in the `web` folder which is supposed to contain the files (HTML, PHP, CSS, etc.) needed for a static website.
That means we need to extract the Podcast Generator files inside the zip to a new folder e.g. `/volume1/web/podcast-generator`. 
By creating a `virtual Host` in the Web Station and referencing that folder, the Podcast generator should become reachable at the specified address podcasts.example.com. In this case we will use Nginx as the web server and PHP 7.3.

![Creating Virtual Host in Web Station](/public/2021-07-19-podcast-generator-on-synology/web-station-virtual-host.png)

So far so good, but there are a couple of stumbling blocks on the way.

## 1. gettext php Extention is missing
With the default PHP settings, the website does not display at all in the browser. 
To investigate errors like this it is usually a good idea to have a look into the Nginx logs. On my Synology Nas, those are located in the 
`/var/log/nginx/` folder (root privileges might be required to access the folder. Running `sudo -i` usually helps).
When checking the nginx `error.log`, we find the following error message: 
`PHP message: PHP Fatal error:  Uncaught Error: Call to undefined function bindtextdomain()`.
After quick googling it turns out that this error is due to the missing gettext extension.

In order to activate the gettext extention, we need to go back to the Web Station inside DSM and change the PHP Settings.
Then dit the profile, find the Extensions section and check the box `gettext` to activate the plugin.
![Creating Virtual Host in Web Station](/public/2021-07-19-podcast-generator-on-synology/web-stationphp-settings.png)


## 2. Files not writable
Now you might be getting some error message when opening the website, that Podcast Generator is unable to write to the folder. To solve this, go to the File Station in Synology, find the `web/podcast-generator`, right click properties and assign some write permissions to the `http` user (not sure if it is best practice, but it works).
![Creating Virtual Host in Web Station](/public/2021-07-19-podcast-generator-on-synology/folder-permissions.png)


## 3. Maximum size of uploaded podcasts

When uploading an mp3 file larger than 32MB, the upload fails. By checking the Nginx logs again, we find the following error message that verifies, indeed the filesize the cause of the issue:
` message: PHP Warning:  POST Content-Length of 41389674 bytes exceeds the limit of 33554432 bytes`

By default, the size of files that can be uploaded through PHP is limited to 32MB. Since podcasts can often be larger, it makes sense to increase the limit. To do this, we need to change the PHP core settings, again via the Webstation. In the `Core` Tab, we need to change the `upload_max_filesize` and `post_max_size` entries, e.g. to `256M` as in the screenshot below.

![Creating Virtual Host in Web Station](/public/2021-07-19-podcast-generator-on-synology/php-max-size-settings.png)

After those settings, the Podcast Generator is finally working as expected.
