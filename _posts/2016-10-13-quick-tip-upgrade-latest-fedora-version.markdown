---
layout: post
title: "Quick tip: Upgrade to the latest Fedora version"
date: 2016-10-13
tags: fedora
---
    sudo dnf upgrade --refresh
    sudo dnf install dnf-plugin-system-upgrade
    # ex. new version is 24
    sudo dnf system-upgrade download --refresh --releasever=24
    sudo dnf system-upgrade reboot
    Source: https://fedoraproject.org/wiki/DNF_system_upgrade