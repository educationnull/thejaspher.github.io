---
layout: post
title:  "Quickly remove comment lines in linux text files"
date:   2012-1-21
categories: Puppet RHEL-CentOS
---

In this walkthrough, we will be doing an install of a Puppet Master, and deploying an Apache server onto a Puppet agent (client).

Puppet Master = puppetserver.example.com

Puppet Agent = apache.example.com

If you do not have DNS in place, you’ll need to edit your /etc/hosts file to allow puppet to resolve server names.

If you have some time, [this guide] helped me set up a BIND server real quick.

Append the following to the hosts file on the Puppet master:

    # The localhost should be listed first.
    172.16.1.100 puppet.example.com puppet
    172.16.1.101 apache.example.com apache

Append the following to the hosts file on the Puppet agent:

    # The localhost should be listed first.
    172.16.1.101 apache.example.com apache
    172.16.1.100 puppetserver.example.com puppetserver

    Verify that hostnames are properly configured by running:
    hostname -f
    hostname -s

Installation
------------

On the Puppet master:

    # Install the EPEL repository
    rpm -ivh https://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-7.noarch.rpm
    # Install the Puppet Server
    yum install puppet-server -y
    # Allow Puppet to communicate over it's default port. 
    # You can remove '-s apache.example.com' to allow any device to communicate on that port.
    iptables -I INPUT -p tcp -s apache.example.com --dport 8140 -j ACCEPT

    service iptables save
    service puppetmaster start
    chkconfig puppetmaster on

On the Puppet agent:

    rpm -ivh https://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-7.noarch.rpm
    # Install the Puppet agent
    yum install puppet -y
    iptables -I INPUT -p tcp -s puppet.example.com --dport 8140 -j ACCEPT
    service iptables save
    service puppet start
    chkconfig puppet on

By default, the Puppet agent looks for the host named “puppet” on your network. Since you’ve already specified this in /etc/hosts, no special configuration is needed.

If you’ve decided to name the Puppet master something else, you’d need to configure your Puppet config file (/etc/puppet/puppet.conf) to reflect that change.

Basic Connectivity
------------------

Let’s start by attempting to pull configuration from agent to the master.

On agent:

    puppet agent --no-daemonize --onetime --verbose

If everything is configured correctly so far, you should see that an SSL certificate request being genereated for your agent. At the end of the output, you should recieve the error:

    Exiting; no certificate found and waitforcert is disabled

What happens here is that by default, the SSL certificate needs to be manually signed by an administrator before configurations can be pulled.

On the Puppet master, check to see that an SSL certificate was generated:

    puppet cert list

You should no

  [this guide]: http://www.broexperts.com/2012/03/linux-dns-bind-configuration-on-centos-6-2/
