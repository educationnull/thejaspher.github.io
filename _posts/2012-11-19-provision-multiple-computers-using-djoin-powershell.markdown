---
layout: post
title:  "Provision multiple computers using djoin + powershell"
date:   2012-11-19
categories: Active-Directory
---
Issue: You need to provision multiple computers and generate blob files for an offline domain join using djoin.exe Solution: Use powershell to loop through an array of computer names and djoin to create the blob file needed. I used an array since I had less than 10 computers to provision. You can parse through a txt or csv file if needed. Since there is no djoin.exe equivalent in powershell, you’d need to pass djoin and it’s arguments like so: `cmd.exe /c djoin [arguments]` This script will create the folder C:\OfflineJoinBlobs and store blobs in there.

    $newComputers = "computerName1", "computerName2", "ComputerName3"
    $OUPath = "`"OU=laptops,OU=computers,OU=user and computers,DC=jaspher,DC=local`""
    $saveLocation = "c:\OfflineJoinBlobs"
    $Result = test-path -path $saveLocation
    if (-not $Result) {
        New-Item $saveLocation -type directory
    }
    foreach ($computer in $newComputers) {
        $cmdString = "djoin /provision /domain jaspher.local /machine " + $computer  + " /machineou " +  $OUPath +   " /savefile  " +  $saveLocation + "\$computer" + "-BLOB.txt"
        cmd.exe /c $cmdString
    }


On the client computer:

    djoin /requestODJ /loadfile -BLOB.txt /windowspath %SystemRoot% /localos