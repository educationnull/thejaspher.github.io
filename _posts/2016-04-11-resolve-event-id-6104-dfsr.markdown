---
layout: post
title: "Resolve Event ID 6104 DFSR"
date: 2016-04-11
tags: active-directory
---
Issue: Event ID 6104 DFSR - The DFS Replication service failed to register the WMI providers. Replication is disabled until the problem is resolved. Fix: Restart the “Windows Management Instrumentation” service either through the gui or command line to see if this resolves the issue. If not, you will need to reregister all the Windows Management Instrumentation (WMI) DLLs. On the problem server:

    CD %windir%\system32\wbem 
    for /f %s in ('dir /b /s *.dll') do regsvr32 /s %s 
    for /f %s in ('dir /b *.mof *.mfl') do mofcomp %s 
    wmiprvse /regserver 

Then restart the “Windows Management Instrumentation” service either through the gui or command line. You should receive an Event ID 6102 DFSR - The DFS Replication service has successfully registered the WMI provider.

[Source](https://social.technet.microsoft.com/Forums/systemcenter/en-US/f6707eec-4b67-4144-9f92-5b7952bcf8f0/the-dfs-replication-service-failed-to-register-the-wmi-providers?forum=operationsmanagergeneral%3Cbr)