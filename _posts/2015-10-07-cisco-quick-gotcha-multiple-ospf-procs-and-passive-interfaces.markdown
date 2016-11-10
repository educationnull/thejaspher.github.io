---
layout: post
title: "[Cisco] Quick gotcha: multiple ospf procs and passive interfaces"
date: 2015-10-07
tags: cisco
---
When running multiple OSPF processes, the `passive-interface` is not respected per process and is applied globally. So the following config is invalid:

    router ospf 1
     passive-interface GigabitEthernet0/0
     passive-interface GigabitEthernet0/1
     network 172.16.0.0 0.0.0.255 area 0
     network 192.168.10.0 0.0.0.255 area 0
    router ospf 10
     passive-interface GigabitEthernet0/0
     network 10.0.0.0 0.0.0.255 area 51
     network 192.168.10.0 0.0.0.255 area 51

The only way is to remove passive interfaces on interfaces that will broadcast any OSPF process.
