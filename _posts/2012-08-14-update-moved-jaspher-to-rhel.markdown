---
layout: post
title:  "Update: Moved jaspher.com from bluehost to RHEL on Amazon ec2"
date:   2012-08-14
categories: meta RHEL-CentOS
---
…or "Hello CentOS!". Follow up: this was pricier than expected. I launched a micro instance of my red hat server, neglecting to research how much it would cost. Micro instances cost $0.02 per hour, but using RHEL, the price jumps to $0.07\. This bring the cost up to around $53 a month, about the same to host this simple blog on bluehost for a year! To minimize the cost of hosting this blog, I started another ec2 instance in Amazon. This time using the CentOS 6.2 (Bare) ami. In short, CentOS is based off of RHEL. A better explanation from nixCraft:

> CentOS Linux is based on the source code of Red Hat Enterprise Linux. However, due to legal issues CentOS can not use Redhat or RHEL trademarked name on their site or on DVD media. CentOS now refers to Red Hat as the “Upstream Vendor” and a “Prominent North American Enterprise Linux Vendor. CentOS has numerous advantages over some of the other clone projects including: an active and growing user community, quickly rebuilt, tested, and QA’ed errata packages CentOS does not provides the RHN style support. It depends upon community mailing lists, forums and 3rd party sites to provide support. CentOS Linux is free but do not get any commercial support or consulting services from Red Hat and lack any software, hardware or security certifications. Also, the CentOS do not get access to Red Hat services like Red Hat Network. If you need access to RHN or support service go for Redhat otherwise CentOS will help you.

I followed the same steps used to move jaspher.com to RHEL, except that I had to:

*   Tar my wordpress installation folder & move it to my new server.
*   Create an SQL dump directly from the mysql prompt.

Both steps can easily be done with a little google-fu. By using a CentOS micro instance, hopefully I can bring down the cost to about $15 a month. This is still double the cost of Bluehost, but the best way for me to learn Linux is by diving in even if it costs a little more.