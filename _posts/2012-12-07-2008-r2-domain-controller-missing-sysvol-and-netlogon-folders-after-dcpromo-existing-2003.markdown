---
layout: post
title:  "2008 r2 Domain Controller missing SYSVOL and NETLOGON folders after dcpromo to existing 2003 domain"
date:   2012-12-07
categories: Active-Directory
---
Issue: After running dcpromo on a 2008 r2 server, the SYSVOL and NETLOGON folders do not appear on the new domain controller. Repadmin shows no errors but dcdiag and best practice analyzer throw out many errors. I already know from experience that I'd need to do a nonauthoritative restore, but just to be sure I used [repadmin /replicate](http://technet.microsoft.com/en-us/library/cc742152(v=ws.10).aspx) from the server that contain the shares DC1, to the server that does not, DC2: `repadmin /replicate DC2 DC1 DC=jaspher,DC=local` but this did not solve the issue. So onto the nonauthoritative restore. On the server that does not have the shares, you'd need to run a nonauthoritative restore. This restore requires you to set the BurFlags registry entry to D2 which signals that it is to pull from a good replica member. Instructions are from the [Microsoft Support](http://support.microsoft.com/kb/290762) site; please go over the entire article before attempting this. On DC2:

1.  Click **Start**, and then click **Run**.
2.  In the **Open** box, type cmd and then press ENTER.
3.  In the **Command** box, type net stop ntfrs.
4.  Click **Start**, and then click **Run**.
5.  In the **Open** box, type regedit and then press ENTER.
6.  Locate the following subkey in the registry: `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\NtFrs\Parameters\Backup/Restore\Process at Startup`
7.  In the right pane, double-click **BurFlags**.
8.  In the **Edit DWORD Value** dialog box, type D2 and then click **OK**.
9.  Quit Registry Editor, and then switch to the **Command** box.
10.  In the **Command** box, type net start ntfrs.
11.  Quit the **Command** box.

The Microsoft article does not point it out but you need to restart the server for the changes to take affect.