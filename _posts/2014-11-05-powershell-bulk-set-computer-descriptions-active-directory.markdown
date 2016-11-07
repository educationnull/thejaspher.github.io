---
layout: post
title: "[Powershell] Bulk set computer descriptions in Active Directory"
date: 2014-11-05
tags: active-directory
---
This script will allow you to fill the Description field in Active Directory. It does so by crawling through your network and pulling information off of the computer using WMI. Unlike the [“popular” method] used, this script does not require you to give Authenticated Users write access to computer objects in Active Directory. This script should be initiated from a Domain Controller by a member of the Domain Admins group.

    ###
    # This script crawls through your network to populate computer descriptions in active directory
    #  with information about the computer (model and service tag).
    # Generated computer descriptions will not overwrite the current description; and if ran more than
    #  once, will skip over computer objects that already have the correct information.
    #
    # $network   - the network part of the address. ex: "10.10.10."
    # $startHost - start at this IP
    # $endHost   - end at this IP
    ###

    #edit your variables here
    $network = "10.10.10."
    $startHost = 70
    $endHost = 150

    function SetDescription($ip)
    {
       # Save time by checking if the host is up first.
      if (Test-Connection $ip -quiet -count 1) {
        echo $("working on: " + $ip)
        
        # Attempt to get service Tag by ip
        $rawServiceTag = Get-WmiObject win32_SystemEnclosure -computername $ip -EA silentlyContinue |  select serialnumber
       
        # Try/ Catch does not work in this situation, so we'll just "Try" by checking if a service tag was returned.
         if (![string]::IsNullOrEmpty($rawServiceTag)) {
             
          # Strip out extra text
          $serviceTag = $rawServiceTag -replace ".*=" -replace "}"
          #echo $serviceTag    

          # Get computer name
          $rawComputerName =  Get-WmiObject -Class Win32_ComputerSystem -computername $ip | select name
          # Strip out extra text
          $computerName = $rawComputerName -replace "---" -replace ".*=" -replace "}"
          #echo $computerName

          # Get computer model
          $rawModel = Get-WmiObject win32_ComputerSystem -computername $ip | select model
          # Strip out extra text
          $model = ($rawModel -replace ".*=" -replace "}").Trim()
          

          $computerInfo = $model + " | " + $serviceTag + " | "
          #echo $computerInfo

          # Get Computer's current description in AD
          $rawCurrentDescription = Get-ADComputer -identity $computerName -Prop description | select Description
          $CurrentDescription = $rawCurrentDescription -replace "---" -replace ".*=" -replace "}"
          #echo $CurrentDescription

            # if ($CurrentDescription -contains $model) {
            if ($CurrentDescription.Contains($model)) {
            echo "-->Description already set by this script"
            }
            else {
            # Insert computerInfo into currentDescription so that you don't overwrite values already inside
            $newDescription = $computerInfo + 

  [“popular” method]: https://4sysops.com/archives/automatically-fill-the-computer-description-field-in-active-directory/