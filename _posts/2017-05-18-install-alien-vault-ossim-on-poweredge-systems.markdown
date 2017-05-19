---
layout: post
title: "Install Alien Vault OSSIM on PowerEdge systems"
date: 2017-05-18
tags: netsec linux
---
Issue: When trying to install OSSIM on a Dell PowerEdge server via USB, you run into this (or similar) error:

    Some of your hardware needs non-free firmware files to operate. 
    The firmware can be loaded from removable media, such as a USB stick or floppy.
    
    The missing hardware files are: bnx/bnx2-mips-09-6.2.1b.fw  
    
    If you have such media available now, insert it, and continue.

This is caused by Debian requiring Broadcom NetXtreme II / bnx2 firmware, which is non-free.

Solution: Add the required .deb package to a new directory called 'firmware' to the root of the USB drive. Assuming the USB is mounted in /mnt, the complete path to the firmware would be /mnt/firmware/firmware-bnx2_0.43_all.deb

Required deb files: <https://packages.debian.org/wheezy/all/firmware-bnx2/download>

Source: <http://lists.us.dell.com/pipermail/linux-poweredge/2016-January/050334.html>
