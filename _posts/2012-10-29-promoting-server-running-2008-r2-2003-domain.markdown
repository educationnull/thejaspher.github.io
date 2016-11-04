---
layout: post
title:  "Promoting a server running 2008 r2 in a 2003 domain"
date:   2012-10-29
categories: Active-Directory
---
Issue: When running dcpromo on a 2008 r2 server in a 2003 domain, you’re prompted to run `adprep /forestprep` from the 2008 r2 installation media. When you run `adprep /forestprep`, you’re prompted that this is only available to existing domain controllers. Solution: You need to run `adprep /forestprep` and `adprep /domainprep` on an existing 2003 domain controller from the 2008 r2 installation media. If your 2003 server is 32-bit, run your need to run adprep32\. Given your installation media is on D:\ if 32-bit:

    d:
    support\adprep\adprep32 /forestprep
    support\adprep\adprep32 /domainprep

if 64-bit:

    d:
    support\adprep\adprep /forestprep
    support\adprep\adprep /domainprep


You should now be able to run dcpromo on your Windows 2008 r2 server.