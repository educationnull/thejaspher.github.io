---
layout: post
title:  "Install Nagios Core on CentOS 6 minimal"
date:   2012-10-15
categories: RHEL-CentOS Monitoring
---
This [script](http://beginlinux.com/blog/2012/01/setup-a-nagios-server-on-centos-6-2/) created by Mike Weber at [BeginLinux.com](http://beginlinux.com/) was designed and tested for Nagios 3.3.1 on CentOS 5.5 and 6.2\. I’ve tested it on a fresh install of CentOS 6.3 minimal and it works perfectly. The modified script (I only changed the version number and download locations):

    #!/bin/bash
    # This script comes with no warranty ...use at own risk
    # Copyright (C) 2010 Mike Weber
    #
    # This program is free software; you can redistribute it and/or modify
    # it under the terms of the GNU General Public License as published by
    # the Free Software Foundation; version 2 of the License.
    #
    # This program is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
    # GNU General Public License for more details.
    #
    # You should have received a copy of the GNU General Public License
    # along with this program or from the site that you downloaded it
    # from; if not, write to the Free Software Foundation, Inc., 59 Temple
    # Place, Suite 330, Boston, MA 02111-1307 USA
    #####################################################################
    # The purpose of the script is to install Nagios and plugins using source.
    # Original script: http://beginlinux.com/blog/2012/01/setup-a-nagios-server-on-centos-6-2/
    # Tested on CentOS 6.3 minimal with Nagios 3.4.1 Core on Oct 12 2012 by Jaspher Respicio
    #####################################################################
    # Script Must Run as root
    if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root" 1>&2
    exit 1
    fi
    cd /usr/local/src
    yum install -y wget
    wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-3.4.1.tar.gz
    wget http://prdownloads.sourceforge.net/sourceforge/nagiosplug/nagios-plugins-1.4.16.tar.gz
    yum install -y httpd php gcc glibc glibc-common gd gd-devel make mysql mysql-devel net-snmp
    useradd nagios
    groupadd nagcmd
    usermod -a -G nagcmd nagios
    tar zxvf nagios-3.4.1.tar.gz
    tar zxvf nagios-plugins-1.4.16.tar.gz
    cd nagios
    ./configure --with-command-group=nagcmd
    make all
    make install; make install-init; make install-config; make install-commandmode; make install-webconf
    cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
    chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
    cd ..
    cd nagios-plugins-1.4.16
    ./configure --with-nagios-user=nagios --with-nagios-group=nagios
    make
    make install
    chkconfig --add nagios
    chkconfig --level 3 nagios on
    chkconfig --level 3 httpd on
    exit 0


In addition to the script, you’ll need to create the password for the default user, nagiosadmin: `htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin` Also, open up port 80 in iptables:

    iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    service iptables save
    service iptables restart


You may also need to set SELinux into permissive mode(Source):

    #See if SELinux is in Enforcing mode.
    getenforce
    #Put SELinux into Permissive mode.
    setenforce 0


Or, instead of disabling SELinux or setting it to permissive mode, you can use the following command to run the CGIs under SELinux enforcing/targeted mode:

    chcon -R -t httpd_sys_content_t /usr/local/nagios/sbin/
    chcon -R -t httpd_sys_content_t /usr/local/nagios/share/


You will need to create an index.html file to avoid an HTTP/1.1 403 Forbidden error: `touch /var/www/html/index.html` To finish up, restart nagios and apache:

    service nagios restart
    service httpd restart

Browse to your installation: http://nagios