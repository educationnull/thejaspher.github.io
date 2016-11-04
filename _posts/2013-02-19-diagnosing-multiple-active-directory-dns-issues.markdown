---
layout: post
title:  "Diagnosing multiple Active Directory DNS issues
date:   2013-02-19
categories: Active-Directory
---
Issue:

Multiple errors from DNS and FRS between 2003 (dc2003) and 2008r2 (dc2008). dcdiag /c shows no errors.

DNS:
dc2008 logs event ID 4013
dc2003 logs event ID 506

File Replication Services:
The following errors happen in sequence on dc2008.
Event ID 13501: ntfrs starts.
Event ID 13562: Could not bind to a Domain Controller.
Event ID 13516: SYSVOL is ready to be shared.
Event ID 13508: The File Replication Service is having trouble enabling replication from dc2003 to dc2008.

The following errors happen in sequence on dc2003.
Event ID 13501: ntfrs starts.
Event ID 13552: The File Replication Service is unable to add this computer to the following replica set: "DOMAIN SYSTEM VOLUME (SYSVOL SHARE)"
Event ID 13555: The File Replication Service is in an error state.

Before anything, I saved and cleared the logs to get a better view of what events trigger what errors.

Troubleshooting steps: DNS

Since FRS issues can stem from DNS, I decided to troubleshoot DNS first.

Doing a net stop dns and net start dns on brings no errors on dc2008 but does bring up event ID 506 on dc2003 twice. Event ID 506 indicates that my SecondaryServers and NotifyServers registry paramaters are corrupted. All I needed to do was to delete these registry entries and run Update Server Data Files on the server. Source.

Restarting the dns service completed successfully without event ID 506 popping up. Event ID 506 DNS: FIXED.

I still have to troubleshoot event ID 4013, but the error has not happened yet, so I'll need to wait to see what event triggers this.

Troubleshooting steps: FRS

Doing a net stop ntfrs and net start ntfrs on brings no errors on dc2008 but does bring up event ID 13552 and 13555 on dc2003. Fixing this was easy. Since dc2003 was the only Domain Controller that was was throwing the error, I needed to to a non-authoritative restore. Source.

After this, event IDs 1553 and 1554 should pop up showing that the replica set was successfully added.

Event IDs 13562, 13508 no longer appear when ntfrs starts up.

Event ID 13552 and 13555 NtFrs: FIXED.

Conclusion

In this case, dns was not the reason for the ntfrs errors. But to solve the ntfrs issues, a non-authoritative restore did the job.