---
layout: post
title:  "Solving: You have exceeded the maximum number of computer accounts you are allowed to create in this domain..."
date:   2013-02-22
categories: Active-Directory
---
Issue: When joining a computer to your domain as a member of the Authenticated Users (in other words, regular user), you receive the following messages:

> The machine account for this computer either does not exist or is unavailable.
> Your computer could not be joined to the domain. You have exceeded the maximum number of computer accounts you are allowed to create in this domain. Contact your system administrator to have this limit reset or increased.
> The following error occurred attempting to join the domain example.com. Your computer could not be joined to the domain. You have exceeded the maximum number of computer accounts you are allowed to create in this domain. Contact your system administrator to have this limit reset or increased.

Solution: You need to change the _ms-DS-MachineAccountQuota_ attribute using ADSI Edit. Steps:

1.  Click Start | Run | and enter adsiedit.msc.
2.  Expand the Domain node and locate the object that begins with “DC=” and contains the domain name of the domain your interested in.
3.  Right on the “DC=” object and click Properties.
4.  Locate the ms-DS-MachineAccountQuota attribute on the Attribute Editor tab and click Edit.
5.  On the Integer Attribute Editor dialog, enter the number of workstations you want users to be able to add. You can enter 0 to prevent users from joining any workstations to the domain or clear the value to remove the limit.
6.  Once you’ve entered the appropriate value, click OK to close the Integer Attribute Editor dialog box and OK again to close the Properties box.
7.  Close ADSI Edit.

This works on 2003, 2008, 2008r2 functional levels [Source from jorink.nl](http://www.jorink.nl/2010/09/increase-the-number-of-workstations-a-user-can-join-to-a-domain/)
