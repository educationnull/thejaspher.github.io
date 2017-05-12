---
layout: post
title: "Install Go and Gophish on CentOS 7"
date: 2017-05-12
tags: netsec linux
---
This walkthrough will install Go, Gophish on a CentOS 7 server.
Gophish (and anyother go apps) will be installed in /opt and will run as a service.
x.x.x.x represents the server's public IP.

Prep

    # Update
    yum -y update
    
    # git and gcc are required to build Gophish. Vim, just becuase.
    yum -y install git gcc vim
    
    # Configure firewall
    systemctl enable firewalld
    systemctl start firewalld
    
    firewall-cmd --zone=public --add-port=80/tcp --permanent
    firewall-cmd --zone=public --add-port=443/tcp --permanent
    firewall-cmd --zone=public --add-port=3333/tcp --permanent
    firewall-cmd --reload
    
Install go
    
    # cd into /tmp for install
    cd /tmp
    
    # Download archive
    curl -LO https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
    
    # Verify download's hash
    sha256sum go1.8*.tar.gz && echo a579ab19d5237e263254f1eac5352efcf1d70b9dacadb6d6bb12b0911ede8994
    
    # Install gophish
    sudo tar -C /usr/local/ -xvzf ./go1.8.1.linux-amd64.tar.gz
    
    # Create Go workspace
    mkdir -p /opt/goapps/{bin,pkg,src}
    
    # Setup path for Go
    sudo vim /etc/profile.d/path.sh
    
    # Add the following line, then save and exit:
    export PATH=$PATH:/usr/local/go/bin
    
    # Configure environment variables
    vim ~/.bash_profile
    
    # Add the following lines, then save and exit:
    export GOBIN="/opt/goapps/bin"
    export GOPATH="/opt/goapps"
    
    # Apply the changes
    source /etc/profile && source ~/.bash_profile
    
    # Test if installed:
    [user@gophish-centos tmp]# go
    'Go is a tool for managing Go source code.
    
    Usage:
    
            go command [arguments]
    
    The commands are:
    
            build       compile packages and dependencies
            clean       remove object files
            doc         show documentation for package or symbol
            env         print Go environment information
            bug         start a bug report
            fix         run go tool fix on packages
            fmt         run gofmt on package sources
            generate    generate Go files by processing source
            get         download and install packages and dependencies
            install     compile and install packages and dependencies
            list        list packages
            run         compile and run Go program
            test        test packages
            tool        run specified go tool
            version     print Go version
            vet         run go tool vet on packages
    
    Use "go help [command]" for more information about a command.
    
    Additional help topics:
    
            c           calling between Go and C
            buildmode   description of build modes
            filetype    file types
            gopath      GOPATH environment variable
            environment environment variables
            importpath  import path syntax
            packages    description of package lists
            testflag    description of testing flags
            testfunc    description of testing functions
    
    Use "go help [topic]" for more information about that topic.
    
Install gophish

    # Install Gophish via 'go get'
    go get github.com/gophish/gophish
    
    # Build Gophish
    cd $GOPATH/src/github.com/gophish/gophish
    go build
    
    # Optional, if this server will be on a VPS, you will need to allow the management
    # interface to be accessible from the public
    vim ./config.json
    
    # Replace: 
    "listen_url" : "127.0.0.1:3333",
    
    # With:
    "listen_url" : "x.x.x.x:3333",
    # Where x.x.x.x is either 0.0.0.0 or the VPS' public IP
    # It may also be a good idea to change the port used to something else too.
    
    # Start Gophish
    ./gophish

In your browser, browse to https://x.x.x.x:3333 to check if everything is working.

Back to the server, press Ctrl+C to exit out of gophish.

Since campains are likely to run for an extended time, you will want gophish to run as a service.

    vim /etc/init.d/gophish
    
    #!/bin/bash
    # /etc/init.d/gophish
    # initialization file for stop/start of gophish application server
    #
    # chkconfig: - 64 36
    # description: stops/starts gophish application server
    # processname:gophish
    # config:/opt/goapps/src/github.com/gophish/gophish/config.json
    
    # define script variables
    
    processName=Gophish
    process=gophish
    appDirectory=/opt/goapps/src/github.com/gophish/gophish
    logfile=/var/log/gophish/gophish.log
    errfile=/var/log/gophish/gophish.error
    
    start() {
        echo 'Starting '${processName}'...'
        cd ${appDirectory}
        nohup ./$process >>$logfile 2>>$errfile &
        sleep 1
    }
    
    stop() {
        echo 'Stopping '${processName}'...'
        pid=$(/usr/sbin/pidof ${process})
        kill ${pid}
        sleep 1 
    }
    
    status() {
        pid=$(/usr/sbin/pidof ${process})
        if [[ "$pid" != "" ]]; then
            echo ${processName}' is running...'
        else
            echo ${processName}' is not running...'
        fi
    }
    
    case $1 in
        start|stop|status) "$1" ;;
    esac

		
Configure the service

    cd /etc/init.d/
    chmod +x gophish
    chkconfig --add gophish
    chkconfig --levels 2345 gophish on
    mkdir /var/log/gophish/
    touch /var/log/gophish/gophish.log
    touch /var/log/gophish/gophish.error

Reboot and browse back to https://x.x.x.x:3333 to confirm that the Gophish starts on startup

Sources
https://gophish.gitbooks.io/user-guide/content/installation.html
https://www.digitalocean.com/community/tutorials/how-to-install-go-1-7-on-centos-7
https://github.com/gophish/gophish/issues/586
