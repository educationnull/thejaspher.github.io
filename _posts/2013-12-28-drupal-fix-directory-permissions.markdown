---
layout: post
title:  "Drupal: Fix directory permissions"
date:   2013-12-28
tags: Drupal
---
Issue: The Security Review module reports that "Drupal installation files and directories (except required) are not writable by the server."

Solution: If you have command line access to your server, use chmod to fix the your permissions.

If your install directory is in: /var/www/drupal/, then:

    chmod 544 -R /var/www/drupal

This will give Drupal sufficent privilege to run, but not to do a few things like install modules - you'd need to install modules and updates using drush (recommended) or wget.