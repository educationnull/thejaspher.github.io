---
layout: post
title: "[Powershell] Run a remote command throughout your /24 network"
date: 2015-08-27
tags: active-directory
---
This script template will help you create a script that runs a command on remote computers in parrallel on your /24 network, or a range within that network. To speed things up even more, the script will not attempt to run the commands if the remote computer is not pingable. You can add extra lines to check if the remote computer is a Window’s system or not.

    # edit your variables here
    $network = "10.10.10."
    $startHost = 1
    $endHost = 254

    workflow TestConnections{ 
        param( $Computers ) Foreach -parallel ($c in $Computers){ 
            if (Test-Connection -ComputerName $c -Count 1 -Delay 1 -EA silentlyContinue){
                DoItALLLLL($c)
            }
            else {
            echo $($c + " is not available!")
        }
        } 
    }

    function DoItALLLLL($ip)
    {
        echo $("working on: " + $ip)
            
        # Run your remote commands here
    }

    $arrComputer = 1..255 | Foreach-Object {  $network + $_ }
    TestConnections -Computers $arrComputer