---
layout: post
title: "Starlabs Starlite 5 Review"
tags: Linux Hardware
---

I Recently bought the Starlite 5, a convertible Linux Tablet from Starlabs systems, a relativly small company from the UK focusing on desinging devices specifically for Linux.
In order to make a judgement if such a device fits your needs, i tried to clearly identify the usage scenarios the device would serve for me. Basically, it is supposed to serve 2 purposes for me:
1. function as a regular tablet: I previiouly owned an android tablet (lenovo tab 10) which was cheap at the time and has aged. I used it for watching videos, browsing, reading e-books, maybe some games. A new hyrid debvice should be suitable for these purposes.
2. use as linux laptop for sutiations where portablility is requred, e.g. quick maintance of servers via ssh, programming etc.


## The Hardware

I am very happy with the build quality. 
It is a bit on the heavy side (for comaprison, the pixel tablet only weights 500 grams (its only a 10 inch screen))

The Display is very sharp and so far always bright enough for my usage. Only thing that would be nice to have is more than 60 herz refresh rate. I really appreciate the 90 hz my Pixel 7 phone, which makes everythink feel just that little bit smooter. One more thing worth noting is that the  Other than that, I can't complain. 

I got the version with th Intel N350, 16 GB DDR5 RAM and 1TB of Storage.

It is even powerful enough to run some small games on Steam.

## Assessoirs

### Keyboard Case

The Keyboard is generally good to type on. I have the issue that sometimes the touchpad i 

### Pen

### Others

The Micro HDMI and Charging Cables work as expected and are of good quality. 


## The Software
I got mine shipped with the Ubuntu 24 Gnome Version. I have not tried any other distros so I can speak only on it. It takes some getting used to but generally it runs very smoothly. E.g. from other touchscreen devices, you are probably used to clicking into a text field, like your browsers adress bar und would exect an onscreen Keyboard to pop up. By default this is not the case on the starlite. instead you need to open the onscreen keybord by swping up from the bottom of the screen. 
Speaking of the touchscreen, the compatibitly is sadly limited for some applications. E.g. I would be intuitive to be able to scroll in VS code by swiping the page up or down but it is not supported. (there are some hacks available but I have so far not been able to get it to work)



## Conclusion

If asked if I could recommend it, I would say yes, even tough the price tag is quite hefty for what you are getting. It really depends on what you are planning to use it for. If it is supposed to replace a regular Android or iOS tablet that iss used for streaming videos, reading, browsing etc. I am pretty sure there are better/cheapter options out there.
If you are looking for what basically is an ultra portable Linux Laptop suitable for more advanced tasks such as programming, document writing or working on a proper command line interface with the option to be used without a keyboard, the Starlite 5 could be something for you :)  

## Problem No. 3: Make Debugging Work in VS Code

You may notice that even when a Breakpoint is set, the debugger does not stop at the specified position. This is because the debugging does not work with Pytest coverage on (As pointed out [here](https://github.com/microsoft/vscode-python/issues/693#issuecomment-367855391)).
This can be fixed by passing the `"--no-cov"` parameter to Pytest.

In total your `lanuch.json` should now contain a configuration like this:

{% highlight json %}
        {
            "name": "Pytest: Debug File",
            "type": "python",
            "request": "launch",
            "module": "pytest",
            "args": [
                "${file}",
                "--no-cov"
            ],
            "console": "integratedTerminal",
            "env": {}
        },
{% endhighlight %}
