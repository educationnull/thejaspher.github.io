---
layout: post
title: "[Powershell] Quickly move FSMO Roles using PowerShell"
date: 2015-07-20
tags: active-directory
---
Requires Powershell v4:


    Move-ADDirectoryServerOperationMasterRole -Identity "Target-DC" -OperationMasterRole SchemaMaster,RIDMaster,InfrastructureMaster,DomainNamingMaster,PDCEmulator

Source - http://social.technet.microsoft.com/wiki/contents/articles/6736.move-transfering-or-seizing-fsmo-roles-with-ad-powershell-command-to-another-domain-controller.aspx
