---
layout: post
title: "Not able to update windows because virtualbox is installed"
---

Recently, I wanted to install a long overdue windows update on an old computer.
running `winver`, we can see the currently installed version of windows.
![Version of Windows Before update](/public/2021-07-08-windows-update-vbox/winver_before.PNG)


## Problem

The update stops at some point due to Virtualbox beeing installed. Removing VB with the windows program management did not solve the issue.


## Solution

Appenently, all Traces of Virtualbox have to be removed for windows to recoginze it as uninstalled.

![Finally, the update continues](/public/2021-07-08-windows-update-vbox/vb_win_update.PNG)
