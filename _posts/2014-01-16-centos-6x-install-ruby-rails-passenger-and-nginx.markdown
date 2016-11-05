---
layout: post
title:  "CentOS 6.x: Install Ruby on Rails with Passenger and Nginx"
date:   2014-01-16
tags: RHEL-CentOS Ruby-on-Rails
---
There are A LOT of resources out there on how to get Ruby on Rails installed on your machine. The problem is that most of them are either for Ubuntu, or conveniently omit the fact that you’d need an actual webserver (nginx) and database (mysql). This walkthrough will show you how to get to your CentOS 6.x server to serve your rails project, using nginx and passenger as your webserver, mysql as your database. Although Ruby 2.1.0 is out, as of the date of this article, the recommended version for rails is still 2.0.0-p353.

Basic setup
-----------

    # Since this is a fresh install, bring your network interface up:
    # If you're on a VPS, the static network interface may already be up and configured with a static IP.
    ifup eth0

    # Run Updates
    yum update -y

    # Install vim for text editing (or any text editor of your preference)
    yum install vim -y

Ruby on Rails
-------------

Install RVM - Ruby Version Manager

    # Download and Install RVM
    curl -L get.rvm.io | bash -s stable

    # Reload profile
    source /etc/profile.d/rvm.sh

    # Download and Install Ruby requirements (gcc-c++, patch, readline-devel, zlib-devel, libyaml-devel, libffi-devel, openssl-devel, autoconf, automake, libtool, bison)
    rvm requirements

Install Ruby

    # Install Ruby 2.0.0-p353 via rvm
    rvm install 2.0.0-p353

    # Use Ruby 2.0.0-p353 by default
    rvm use 2.0.0-p353 --default

    # Update gems
    rvm rubygems current

Install Rails

    # Install sqlite-devel, required by Rails, not installed via the wizard
    # Although we won't be using SQLite as our database, the installation fails without
    yum install sqlite-devel -y

    # A javascript library is required for Rails, install node.js
    yum install nodejs -y

    # Do not install documentation for each package - this will save time and space
    echo "gem: --no-ri --no-rdoc" > ~/.gemrc

    # Install Rails, this will take a while
    gem install rails

Servers
-------

Install Passenger and Nginx

    # Install Passenger
    gem install passenger

    # Curl-devel is requred to download nginx using rvm
    yum install curl-devel -y

    # Install nginx using rvmsudo. Take note of the symbol used: tick (`) not a single quote(')
    # The --auto and --auto-download options will run the script with the default options:
    # The --prefix=/opt/nginx option places nginx into the default directory recommended by the script. 
    rvmsudo `which passenger-install-nginx-module` --auto --auto-download --prefix=/opt/nginx

Configure Nginx Use vim /etc/init.d/nginx to create the start/restart/stop script for nginx:

    #!/bin/sh
    #
    # nginx - this script starts and stops the nginx daemon
    #
    # chkconfig:   - 85 15 
    # description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
    #               proxy and IMAP/POP3 proxy server
    # processname: nginx
    # config:      /usr/local/nginx/conf/nginx.conf
    # pidfile:     /usr/local/nginx/logs/nginx.pid

    # Source function library.
    . /
