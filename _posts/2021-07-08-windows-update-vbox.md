---
layout: post
title: "Not able to update windows because virtualbox is installed"
---

Recently, I wanted to install a long overdue windows update on an old computer.
running `winver`, we can see the currently installed version of windows.
![Version of Windows Before update](/public/2021-07-08-windows-update-vbox/winver_before.PNG)


## Problem
The update stops at some point due to Virtualbox beeing installed. Removing VB with the windows program management did not solve the issue.
![Error when updating windows](/public/2021-07-08-windows-update-vbox/virtualbox-message.jpg)


## Solution
Appenently, all Traces of Virtualbox have to be removed for windows to recoginze it as uninstalled.

This is the list of locations where files related to Virtualbox are stored:
 - C:\Users\dechert\VirtualBox VMs
 - C:\Users\dechert\.VirtualBox
 - C:\ProgramData\VirutalBox
 - C:\Program Files\Oracle\VirtualBox Guest Additions

Also delete all files starting with `VBox` in `C:\Windows\System32\drivers`.
I am not 100% sure it is nessesary to remove the Guest Additions but I did so anyway.

Some deletions also need to be made in the Registry, [as described here](https://dottech.org/101997/how-to-uninstall-virtualbox-drivers-on-windows/).

![Finally, the update continues](/public/2021-07-08-windows-update-vbox/vb_win_update.PNG)
