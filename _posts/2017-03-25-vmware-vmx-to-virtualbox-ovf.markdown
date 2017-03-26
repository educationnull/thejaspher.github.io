---
layout: post
title: "Convert VMWare VMX to VirtualBox OVF"
date: 2017-03-25
tags: homelab
---

Requires VMWare Player to be installed. In an elevated command prompt:

    cd \to-your-vm-path
    "C:\Program Files (x86)\VMware\VMware Player\ovftool\ovftool.exe" .\sourceVM.vmx .\exportVM-vbox.ovf
