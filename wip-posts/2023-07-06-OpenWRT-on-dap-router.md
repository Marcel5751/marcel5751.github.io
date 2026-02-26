---
layout: post
title: "How-To Flash and configure OpenWRT on D-Link DAP-2695"
---

This guide describes how to install the OpenWRT firmware on a D-Link DAP-2695 Access Point. The process will be (slightly) different for other models.

## Flash OpenWRT firmware
 - Download openWRT image for the Router, in this case: https://openwrt.org/toh/d-link/dap-2695
 - Download and install a TFTP server software e.g. [Open TFTP Server](https://sourceforge.net/projects/tftp-server/),. This is needed to transfer the firmware image archive to the Router
 - Place firmware image in C:\OpenTFTPServer folder (default)
 - Start the TFTP server by running the OpenTFTPServerMT.exe located in the C:\OpenTFTPServer folder
 - Enable SSH in Router web interface (don't forget to save settings)
![Enable SSH in OEM router config](/public/2023-07-06-OpenWRT-on-dap-router/turn-on-ssh.png)

 - Use putty to connect (ssh on CLI does not work for some reason)
![SSH into Router with putty](/public/2023-07-06-OpenWRT-on-dap-router/putty1.png)

 - Enter the password for the original router configuration

Once logged in to the Router command line interface, run the following command (example of NETWORK_IP_ADDRESS_OF_DEVICE: 192.168.10.110)

```shell
tftp getfirmware openwrt-22.03.5-ath79-generic-dlink_dap-2695-a1-squashfs-factory.img <NETWORK_IP_ADDRESS_OF_DEVICE>
```

 - Something like "Down! Please reboot device!" should be the response.
 - After the reboot, OpenWRT should be installed. 

## Connect Router to Network

OpenWRT will configure the router with the default IP of 192.168.1.1. If your computer is connected to another subnet, e.g. its on the `.10` subnet, the computer will not be able to access the router (neither ping, nor accessing the Web Interface via browser will work). A little workaround is required:

 - Connect the computer to the Router via Ethernet cable plugged into the "LAN" (not WAN plug) of the Router. This is necessary because Wifi is disabled by default
 - Change the Ethernet adapter configuration of our computer, we can connect to the router

![IP configuration](/public/2023-07-06-OpenWRT-on-dap-router/ip-config.png)

Configuration of the Router can now be opened in a browser by accessing http://192.168.1.1

The IP address of the Router needs to be changed to be within the network address range e.g. 192.168.10.130. Settings should look like this:
![IP configuration](/public/2023-07-06-OpenWRT-on-dap-router/configure-static-ip-of-router.png)

If a custom DNS server, e.g. for PI-hole should be used, this needs to be configured as well.

![IP configuration](/public/2023-07-06-OpenWRT-on-dap-router/openwrt-dns-setting.png)

Now we can unplug the Ethernet cable that connects our device and the Router and connect the router's WAN port with an existing Switch in the Network.

## Setup Wifi Access Point

By default, the Wireless adapter is disabled in OpenWRT. In order to use the Router as an Access Point to the network, further configuration is needed.

### Installing the wpad package

The pre-installed package `wpad-basic-wolfssl`, as you can see in the package description, lacks support for 802x / RADIUS authentication. Therefore we need to remove it to avoid conflicts before installing the fully-featured package. We can manage packages with the preinstalled `opkg` package manager.

```
opkg remove wpad-basic-wolfssl
```

Before OpenWRT is able to download a new package, we need to run opkg update. If it fails, due to a missing internet connection of the router itself, this is most likely due to a missing DNS setting, so make sure that the DNS was configured as described in the step above (for more information see here: https://openwrt.org/docs/guide-quick-start/checks_and_troubleshooting). If it is still not working, simply retrying the opkg command can help.

Afterward, we can install the fully-featured [wpad package](https://openwrt.org/packages/pkgdata/wpad) (installation of the wolfssl version did not work for some reason).

```
opkg install wpad
```

After installation, a reboot is required. We can just type "reboot" into the CLI.

Optional (if you are not a fan of vim):
```
opkg install nano
```

## Configuring Wifi Adapter for RADIUS Server

We can now edit the wifi configuration. There are 2 different configurations here: the wifi devices and the wifi interfaces. For more details, please refer to the [official OpenWRT guide](https://openwrt.org/docs/guide-user/network/wifi/wireless.security.8021x?s[]=radius#basic_8021x_wireless_user_authentication): 

Still in the ssh session, run run `nano /etc/config/wireless`

The configuration should look something like this:

```
config wifi-device 'radio0'
        option type 'mac80211'
        option path 'pci0000:00/0000:00:00.0'
        option channel 'auto'
        option band '5g'

config wifi-iface 'radio0'
        option device 'radio0'
        option mode 'ap'
        option ssid 'swagwifi-5g'
        option network 'lan'
        option encryption 'wpa2'
        option server '192.168.10.12'
        option port '1812'
        option key '<RADIUS_SHARED_KEY>'

config wifi-device 'radio1'
        option type 'mac80211'
        option path 'platform/ahb/18100000.wmac'
        option channel 'auto'
        option band '2g'

config wifi-iface 'radio1'
        option device 'radio1'
        option mode 'ap'
        option ssid 'swagwifi'
        option network 'lan'
        option encryption 'wpa2'
        option server '192.168.10.12'
        option port '1812'
        option key '<RADIUS_SHARED_KEY>'
```


The Router needs to be rebooted for the changes to take effect.

For more information, see this guide: https://openwrt.org/docs/guide-user/network/wifi/wireless.security.8021x?s[]=radius#basic_8021x_wireless_user_authentication

If the configuration contains a mistake, e.g. a `-` in the name, which apparently is not allowed there is no error message, instead the "Wireless" option below the "Interfaces" will disappear from the Network menu.

![missing config](/public/2023-07-06-OpenWRT-on-dap-router/wifi-settings-gone.png)

To get information about the error, the following command can be used:

```shell
root@OpenWrt:~# uci show wireless > /dev/null
uci: Parse error (invalid character in name field) at line 23, byte 35
```

Reboot the router after fixing the error.

