---
layout: post
title: "[CentOS 7] Installing Ruby on Rails"
date: 2015-11-09
tags: rhel-centos ruby-on-rails
---
I’ve been playing around with Ruby on Rails recently and have grown to appreciate the framework and the [MVC Paradigm]. I’ve been playing around with various cloud IDEs like [Cloud9]and [Nitrous], but nothing beats a local dev server and Vim. I used this [Digital Ocean] tutorial as a starting point, with a few tweaks. I’ll be using [rbenv]to manage my Ruby installations. [RVM]is an alternative to this.

Install Ruby dependencies
-------------------------

    sudo yum install -y git-core zlib zlib-devel gcc-c++ patch readline readline-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison curl sqlite-devel

Install rbenv from git
----------------------

    cd
    git clone git://github.com/sstephenson/rbenv.git .rbenv
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    exec $SHELL

    git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
    exec $SHELL

    # Don't forget to reload your bash profile!
    .  ~/.bash_profile

Install Ruby and prep for Rails
-------------------------------

Use `rbenv install -l` to list available Ruby versions. As of this post, 2.2.3 is the latest supported version.

    # Install 2.2.3
    rbenv install -v 2.2.3
    # Use 2.2.3 globally
    rbenv global 2.2.3

Insert the following into your gemrc file to skip local installation of documentation for gems:

    echo "gem: --no-document" > ~/.gemrc

Install the bundler gem:

    gem install bundler

Install Rails
-------------

Install the latest version of Rails:

    gem install rails
    rbenv rehash

Install the nodejs runtime:

    sudo yum -y install epel-release
    sudo yum -y install nodejs

At this point, you should have RoR installed. From here, you’d want to install the database of your choice (postgres, maridab, etc) before generating your new project.

Configure your firewall
-----------------------

If this server is remote and not local, you’’ll need to allow tcp port 3000 through your firewall as this is the default port Rails uses:

    firewall-cmd --zone=public --add-port=3000/tcp --permanent
    firewall-cmd --reload

Since this is on a remote server, remember to bind to the correct IP when running `rails server`. For example, if your server is on IP 10.0.0.200, then to run the Rails server you’ll type: <code>rails s -b 10.0.0.200</c>

  [MVC Paradigm]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
  [Cloud9]: https://c9.io/
  [Nitrous]: https://www.nitrous.io/
  [Digital Ocean]: https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-centos-7
  [rbenv]: https://github.com/sstephenson/rbenv
  [RVM]: http://kgrz.io/2014/02/04/Programmers-guide-to-choosing-ruby-version-manager.html