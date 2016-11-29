---
layout: post
title: "[nginx] Better robots.txt management"
date: 2015-01-23
tags: linux
---
Robots.txt is a file that lets legitimate crawlers know what pages/ directories they are allowed to scan. Ideally, you’d want to disallow areas that are reserved for backend management like ‘/admin/’, while allowing crawlers to access public areas of your site.
Here’s Drupal’s default robots.txt:

    #
    # robots.txt
    #
    # This file is to prevent the crawling and indexing of certain parts
    # of your site by web crawlers and spiders run by sites like Yahoo!
    # and Google. By telling these "robots" where not to go on your site,
    # you save bandwidth and server resources.
    #
    # This file will be ignored unless it is at the root of your host:
    # Used:    http://example.com/robots.txt
    # Ignored: http://example.com/site/robots.txt
    #
    # For more information about the robots.txt standard, see:
    # http://www.robotstxt.org/robotstxt.html
    #
    # For syntax checking, see:
    # http://www.frobee.com/robots-txt-check

    User-agent: *
    Crawl-delay: 10
    # Directories
    Disallow: /includes/
    Disallow: /misc/
    Disallow: /modules/
    Disallow: /profiles/
    Disallow: /scripts/
    Disallow: /themes/
    # Files
    Disallow: /CHANGELOG.txt
    Disallow: /cron.php
    Disallow: /INSTALL.mysql.txt
    Disallow: /INSTALL.pgsql.txt
    Disallow: /INSTALL.sqlite.txt
    Disallow: /install.php
    Disallow: /INSTALL.txt
    Disallow: /LICENSE.txt
    Disallow: /MAINTAINERS.txt
    Disallow: /update.php
    Disallow: /UPGRADE.txt
    Disallow: /xmlrpc.php
    # Paths (clean URLs)
    Disallow: /admin/
    Disallow: /comment/reply/
    Disallow: /filter/tips/
    Disallow: /node/add/
    Disallow: /search/
    Disallow: /user/register/
    Disallow: /user/password/
    Disallow: /user/login/
    Disallow: /user/logout/
    # Paths (no clean URLs)
    Disallow: /?q=admin/
    Disallow: /?q=comment/reply/
    Disallow: /?q=filter/tips/
    Disallow: /?q=node/add/
    Disallow: /?q=search/
    Disallow: /?q=user/password/
    Disallow: /?q=user/register/
    Disallow: /?q=user/login/
    Disallow: /?q=user/logout/

This is good enough in many cases, but sometimes you may want to make changes as the site grows in scope. For example, you want to disallow ALL areas of your website (good config for internal usage websites that are open to the web):

    User-agent: *
    Disallow: /

The issue is that the robots.txt file is replaced every time you do a core update to Drupal, so all that hard work is gone after the next core update (this really depends on how you manage code).

One problem (minor, does not affect site’s behavior) we run into is with the Hacked! module. Your reports will give you a warning that a core file has changed. This is a very serious warning that was triggered by a small change. It is also best practices to never ‘hack’ core files and modules.

There is a really good module that solves this by allowing you to save a custom robots.txt in the module, and using hook_menu to call that file. For this to work, robots.txt must be manually removed from core. But the issue remains - robots.txt is replaced after a core update, rendering the custom changes useless until you remember to delete the file. Also, from my understanding, Hacked! will also issue you a warning that a module file has been changed.

The best solution imo, is to do this at the server level. If you're using nginx, you can issue an alias to point robots.txt to your custom file that lives outside of drupal core. Here's how that would look like:

```
location = /robots.txt {
     allow all;
     log_not_found off;
     access_log off;
     alias /var/www/rewrites/robots.txt;
}
```

This snippet would be placed in your default.conf file, in the server block.

By using this method, you keep core's robots.txt where it is. No warnings are displayed in Hacked! and you do not need to remove/ edit core files.
