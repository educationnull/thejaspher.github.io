---
layout: post
title: "[Bash] Dirty trick - quick fixes on your dev server"
date: 2015-09-16
tags: drupal linux
---
On my local dev server, I have the following setup:

-   CentOS 7
-   httpd
-   mariadb
-   All dev sites located in a subdirectory in /var/www/html/

Occasionally things will break, or permissions need to be reset on new dev sites. Here’s a quick fix that I use to get all the usual troubleshooting steps out of the way. Note that this is on a local dev server, you’d never blindly do this on a prod server.

Insert the following into your .bashrc file: `vim ~/.bashrc`

    alias fix='chcon -R -t httpd_sys_rw_content_t /var/www/html/*/sites/default; chmod -R 755 /var/www/html/; chown -R apache:apache /var/www/html/; systemctl restart httpd; systemctl restart mariadb'

Then reload bash : `. ~/.bashrc`

The above alias does the following

-   `alias fix='...' `creates an alias called fix. This allows you to run the fix command to run troubleshooting steps.
-   `chcon -R -t httpd_sys_rw_content_t /var/www/html/*/sites/all/default;` Configures selinux to allow apache to make changes on the filesystem, specifically the default folder in drupal.
-   `chmod -R 755 /var/www/html/`; Change subdirectory and file permissions to 755
-   `chown -R apache:apache /var/www/html/;` Gives apache ownership to all required site files
-   `systemctl restart httpd; systemctl restart mariadb` Restarts apache and mariadb

Ideally, you’d want to use something like puppet on your dev server, especially if you need to support multiple servers.

Another thing that I can’t be bothered with while in dev mode - confirming to `drush` that I really want to execute the command I just typed in. You can use `drush -y [blah blah]` but my pinky needs to rest sometimes.

Add to your .bashrc: `alias drush='drush -y'` and your pinky will thank you.
