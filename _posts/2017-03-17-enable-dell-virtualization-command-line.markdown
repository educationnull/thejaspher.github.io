---
layout: post
title: "Dell Server - Enable virtualization via command line"
date: 2017-03-17
tags: sysadmin
---
    C:\> cd \Program Files\Dell\SysMgt\oma\bin
    C:\Program Files\Dell\SysMgt\oma\bin> .\omconfig.exe chassis biossetup attribute=cpuvt setting=enabled
    BIOS setup configured successfully. Change will take effect after the next reboot.

sources
- http://sogoth.com/?p=430
- http://www.couradical.com/updates/2013/4/1/remote-enabling-bios-features-using-openmanage
