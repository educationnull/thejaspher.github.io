---
layout: post
title: "[Fedora] Quick tip: Upgrade to the latest Fedora version"
date: 2016-10-13
tags: linux
---
    sudo dnf upgrade --refresh
    sudo dnf install dnf-plugin-system-upgrade
    # ex. new version is 26
    sudo dnf system-upgrade download --refresh --releasever=26
    sudo dnf system-upgrade reboot
    Source: https://fedoraproject.org/wiki/DNF_system_upgrade
