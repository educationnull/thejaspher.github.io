---
layout: post
title:  "[RHEL /CentOS 7] Apache cannot send emails on CentOS"
date:   2014-05-16
tags: linux
---
Issue: Drupal cannot send email. Stack is on LAMP, distro is CentOS.

Cause: SELinux is blocking emails being sent out. To confirm, 
    sestatus -b | grep sendmail. 
	
If `httpd_can_sendmail` is set to 'off', then that's the cause.

Solution:
Do NOT just turn SELinux off. Run `setsebool httpd_can_sendmail=1` to allow emails to be sent out.
