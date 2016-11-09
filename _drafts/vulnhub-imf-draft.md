
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

A possible vulnerability in web front-end is the contact form. The contact form does not display any errors after submission and Burpsuite shows nothing interesting. This could be a dead end.

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
imfadministratorbase64
```

Okay, back to the browser with our findings:

- http://172.16.0.76/imfadministratorbase64 - 404
- http://172.16.0.76/imfadministrator - 200 OK

Awesome, found the admin login area.

I tried some dummy data for username and password. Two things I noticed about this form:

1. It looks like the form insecurely lets me know that the username I've tried is incorrect. 
2. Uses a phpsessid.

In burpsuite, the "failed login" page's response gives us a nice note from the develoer (also could probably be Roger S. Michaels - the director):
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

In burpsuite, I tested the following usernames to see if if the "Invalid username" message does not appear:

    rmichaels@imf.local: Invalid username.
    akeith@imf.local: Invalid username.
    estone@imf.local: Invalid username.
    *** rmichaels: Invalid password. ***
    akeith: Invalid username.
    estone: Invalid username.
    
Got a hit on Roger S. Michaels's username. 

### Bruteforce

I'll try to brute force Roger's account using hydra and a list built into kali at `/usr/share/wordlists/*`. I'll start with the popular rockyou list. I was able to craft a http-form-post string in hydra thanks to Burpsuite and [this tutorial](http://insidetrust.blogspot.com/2011/08/using-hydra-to-dictionary-attack-web.html).

```bash
cd /usr/share/wordlists/
gunzip ./rockyou.txt.gz

hydra 172.16.1.76 http-form-post "/imfadministrator:user=^USER^&pass=^PASS^:Invalid password" -l rmichaels -P /usr/share/wordlists/rockyou.txt
```
The problem I have now is that I keep on getting different results when I run and a whole bunch of false positives. This could be brute force protection behind the server or the login attempts are coming in too fast for the server to respond with the correct "Invalid password" string.  After throttling the threads down to 1, I still get false positives. I'm guessing there's some type of protection against brute forcing. I need to see the http response when hydra is being ran. It looks like I should be able to use Hydra through a proxy, but I can find solid documenation on how to do that from the command line. So I'll just fire up Wireshark to follow the http streams. Here's the conversation:

```
GET /imfadministrator HTTP/1.0
Cookie: PHPSESSID=8qicfn6g1dneig1p7ro06i10g4
Host: 192.168.100.76
User-Agent: Mozilla/5.0 (Hydra)

HTTP/1.1 301 Moved Permanently
Date: Wed, 09 Nov 2016 00:31:35 GMT
Server: Apache/2.4.18 (Ubuntu)
Location: http://192.168.100.76/imfadministrator/
Content-Length: 327
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://192.168.100.76/imfadministrator/">here</a>.</p>
<hr>
<address>Apache/2.4.18 (Ubuntu) Server at 192.168.100.76 Port 80</address>
</body></html>

```

So it looks like I'm being given a 301 error when logging in through hydra. Possibly being blocked due to Hydras user-agent.
```
hydra -l rmichaels -P /usr/share/wordlists/rockyou.txt 192.168.100.76 http-form-post "/imfadministrator:user^USER^&pass=^PASS^:Invalid:H=Cookie: PHPSESSID=8qicfn6g1dneig1p7ro06i10g4; User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 9_2_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13D15 Safari/601.1" -t 1
```
