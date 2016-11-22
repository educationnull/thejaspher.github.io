---
layout: post
title: "[CentOS 7]: install and configure openVPN server"
date: 2015-11-06
tags: linux netsec
---
DigitalOcean has a [good tutorial] on how to set up an OpenVPN server on CentOS 7. I was not able to follow step-by-step since the tutorial goes into setting up routing using iptables, instead of using the builtin firewall-cmd. Later, I had issues connecting to the server from a Windows machine, but was able to resolve that (included below).

Here’s an abriged version of the DigitalOcean tutorial, with the fixes.

Install OpenVPN
---------------

    yum install epel-release
    yum install openvpn easy-rsa

Configure OpenVPN
-----------------

    cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf /etc/openvpn

    vim /etc/openvpn/server.conf

    # These are the default values for fields
    # which will be placed in the certificate.
    # Don't leave any of these fields blank.
    export KEY_COUNTRY="US"
    export KEY_PROVINCE="NY"
    export KEY_CITY="New York"
    export KEY_ORG="DigitalOcean"
    export KEY_EMAIL="sammy@example.com"
    export KEY_OU="Community"
    ...
    # X509 Subject Field
    export KEY_NAME="server"
    ...
    export KEY_CN=openvpn.example.com

Generating Keys and Certificates
--------------------------------

    mkdir -p /etc/openvpn/easy-rsa/keys
    cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa
    cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
    cd /etc/openvpn/easy-rsa
    source ./vars
    ./clean-all
    ./build-ca
    ./build-key-server server
    ./build-dh
    cd /etc/openvpn/easy-rsa/keys
    cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
    cd /etc/openvpn/easy-rsa
    ./build-key client

Routing using firewall-cmd
--------------------------

    firewall-cmd --permanent --add-service openvpn
    firewall-cmd --permanent --add-masquerade
    firewall-cmd --reload

Add the following line to `sysctl.conf`

    vim /etc/sysctl.conf

    net.ipv4.ip_forward = 1

    systemctl restart firewalld
    systemctl restart network.service

Configuring your Windows client
-------------------------------

Download the following lines from your server to your client:

-   /etc/openvpn/easy-rsa/keys/ca.crt
-   /etc/openvpn/easy-rsa/keys/client.crt
-   /etc/openvpn/easy-rsa/keys/client.key

For this example, we’ll save it locally to `c:\openvpn\certs`

Use notepad (or similar) to create a new file called client.ovpn with the following contents:

    client
    dev tun
    proto udp
    remote your_server_ip 1194
    resolv-retry infinite
    nobind
    persist-key
    persist-tun
    comp-lzo
    verb 3
    ca c:\\openvpn\\certs\\ca.crt
    cert c:\\openvpn\\certs\\client.crt
    key c:\\openvpn\\certs\\client.key

Replace `your_server_ip` with the public IP of the server, and make sure to use double backslashes

  [good tutorial]: https://www.digitalocean.com/community/tutorials/how-to-setup-and-configure-an-openvpn-server-on-centos-7
  
