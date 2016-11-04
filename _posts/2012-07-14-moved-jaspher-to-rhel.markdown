---
layout: post
title:  "Moved jaspher.com from bluehost to RHEL on Amazon ec2"
date:   2012-07-14
tags: meta RHEL-CentOS
---
In an effort to better understand the Linux OS, I’ve moved my personal blog over from the shared hosting solution Bluehost to Amazon’s EC2 service.

If I have time, I’ll go through this in detail. At the moment I’m studying for my MS 70-640 so I’ll just dump a few links that helped me with this transition, along with general steps.

Issue: You want to move your wordpress installation from a shared server to your own RHEL 6 server.

Solution (setup):

Spin up a new ec2 instance, there are many tutorials online to get you started.
Install/ configure apache:yum install httpd (more info).
Potential gotcha – you need to open up port 80 in your ec2 security group AND your RHEL instance.

    # To reconfigure your firewall on RHEL: 
    system-config-firewall-tui
    # Install mysql: 
	yum install mysql-server mysql
    # Install php-mysql: 
	yum install -y php php-mysql
    # Install php: 
	yum install php


Migration:

Export my wp database on bluehost to an SQL file.
Create a new databse and import the SQL dump into your new database.
Zip your wordpress files up in bluehost and import/unzip that into you /var/www/html folder (or whatever you specified your DocumentRoot to be).


Other things I did:

Since I’m planning on hosting other sites on this server, I’m implementing VirtualHosts. You’d need to configure httpd.conf. More info here and here.

Ran into “DocumentRoot does not exist”? fix here and here.

Enable permalinks/ clean urls: Since I moved everything from my previous working installation, my .htaccess file was already configured correctly. What I forgot to do was to add another directive in my httpd.conf file (since I’m using VirtualHosts.) I added the following:

    AllowOverride all
