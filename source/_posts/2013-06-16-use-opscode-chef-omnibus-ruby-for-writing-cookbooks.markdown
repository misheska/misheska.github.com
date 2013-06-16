---
layout: post
title: "Use Opscode Chef Omnibus Ruby for writing cookbooks"
date: 2013-06-16 02:35
comments: true
categories: chef
---
* list element with functor item
{:toc}

If your only use of Ruby is because you want to write cookbooks for Opscode
Chef, rather than going through the bother of setting up a full Ruby
development environment via [rvm](http://misheska.com/blog/2013/06/16/using-rvm-to-manage-multiple-versions-of-ruby/) or [rbenv](http://misheska.com/blog/2013/06/15/using-rbenv-to-manage-multiple-versions-of-ruby/)
you can just reuse the Ruby 1.9.x environment that is bundled with the
Opscode Omnibus Chef installer.

In this post, I'll cover how to configure the Ruby 1.9.x interpreter bundled
with the Opscode Omnibus Chef installer on Mac OS X, Linux and Windows so
it can be used for writing cookbooks.

How to set up Omnibus Chef Ruby on Mac OS X
===========================================
First install Apple Xcode, which includes a C compiler needed to build the
tools required for cookbook development from source (like
[Berkshelf](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/) ).
Download and install the latest version of Xcode from the App Store, if you
don't have it installed already.  Also make sure you install the *Command Line
Tools* by choosing the menu item <code>Xcode -> Preferences...</code> and click
on the *Downloads* tab.  Click on the *Install* button to download the
Command Line Tools.
![Xcode Command Line Tools](/images/xcodecommandline.png)

In a web browser, go to the <http://www.opscode.com/chef/install> page to
display the instructions for installing the Chef Client via the Opscode
Omnibus Chef installer.

As of this writing, the quick install instructions for Mac OS X are as
follows:

    $ curl -L https://www.opscode.com/chef/install.sh | sudo bash

The Chef Client installer also installs Ruby 1.9.3 for its own use in
the directory `/opt/chef/embedded`.  You can also use this copy of Ruby
for your own cookbook development.

WARNING: Don't try to mix and match the Chef Client's Ruby 1.9.3 together
with a RVM/rbenv Ruby development setup.  Choose one or the other.  If your
Ruby needs go beyond Chef and writing Chef cookbooks, set up a "real"
RVM/rbenv Ruby environment.

Now that you have a Ruby 1.9.3 environment via the Chef Client install, you
can install any extra gem dependencies needed for writing Chef cookbooks by
using the `/opt/chef/embedded/bin/gem install` command.  For example, here's
how to install Berkshelf, a popular cookbook authoring support tool:

    $ sudo /opt/chef/embedded/bin/gem install berkshelf --no-ri --no-rdoc
    Password:
    Fetching: i18n-0.6.1.gem (100%)
    Fetching: multi_json-1.7.7.gem (100%)
    Fetching: activesupport-3.2.13.gem (100%)
    ...
    Successfully installed safe_yaml-0.9.3
    Successfully installed test-kitchen-1.0.0.alpha.7
    Successfully installed berkshelf-2.0.3
    43 gems installed

Create a soft link to any gem-installed binaries in an existing PATH directory,
like `/usr/local/bin`

    # On Mac OS X, /usr/local/bin is not created by default
    $ sudo mkdir -p /usr/local/bin
    Password:
    $ sudo ln -s /opt/chef/embedded/bin/berks /usr/local/bin/berks
    $ berks -v
    Berkshelf (2.0.3)

Don't be tempted to add `/opt/chef/embedded/bin` to your PATH.  You still want
to keep Opscode's Ruby install separate from your main system Ruby install.

How to set up Omnibus Chef Ruby on Linux
========================================
Make sure all the prerequisite packages are installed for the gems you will
be using.

Ubuntu prerequisites:

    $ sudo apt-get update
    $ sudo apt-get install -y curl
    $ sudo apt-get install -y build-essential git
    $ sudo apt-get install -y libxml2-dev libxslt-dev libssl-dev

RHEL/CentOS prerequisites:

    $ sudo yum update
    $ sudo yum install -y curl
    $ sudo yum install -y git
    $ sudo yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel
    $ sudo yum install -y libyaml-devel libffi-devel openssl-devel make bzip2
    $ sudo yum install -y autoconf automake libtool bison
    $ sudo yum install -y libxml2-devel libxslt-devel

In a web browser, go to the <http://www.opscode.com/chef/install> page to
display the instructions for installing the Chef Client via the Opscode
Omnibus Chef installer for your distribution of Linux.

As of this writing, the quick install instructions for Ubuntu/CentOS are as
follows:

    $ curl -L https://www.opscode.com/chef/install.sh | sudo bash

The Chef Client installer also installs Ruby 1.9.3 for its own use in
the directory `/opt/chef/embedded`.  You can also use this copy of Ruby
for your own cookbook development.

WARNING: Don't try to mix and match the Chef Client's Ruby 1.9.3 together
with a RVM/rbenv Ruby development setup.  Choose one or the other.  If your
Ruby needs go beyond Chef and writing Chef cookbooks, set up a "real"
RVM/rbenv Ruby environment.

Now that you have a Ruby 1.9.3 environment via the Chef Client install, you
can install any extra gem dependencies needed for writing Chef cookbooks by
using the `/opt/chef/embedded/bin/gem install` command.  For example, here's
how to install Berkshelf, a popular cookbook authoring support tool:

    $ sudo /opt/chef/embedded/bin/gem install berkshelf --no-ri --no-rdoc
    Password:
    Building native extensions.  This could take a while...
    Fetching: httpclient-2.2.0.2.gem (100%)
    Fetching: rubyntlm-0.1.1.gem (100%)
    Fetching: uuidtools-2.1.4.gem (100%)
    ...
    Successfully installed safe_yaml-0.9.3
    Successfully installed test-kitchen-1.0.0.alpha.7
    Successfully installed berkshelf-2.0.3
    25 gems installed

Create a soft link to any gem-installed binaries in an existing PATH directory,
like `/usr/local/bin`

    $ sudo ln -s /opt/chef/embedded/bin/berks /usr/local/bin/berks
    $ berks -v
    Berkshelf (2.0.3)

Don't be tempted to add `/opt/chef/embedded/bin` to your PATH.  You still want
to keep Opscode's Ruby install separate from your main system Ruby install.

How to set up Omnibus Chef Ruby on Windows
==========================================
In a web browser, go to the <http://www.opscode.com/chef/install> page to
display the instructions for installing the Chef Client via the Opscode
Omnibus Chef installer for your distribution of Linux.

After you select a Chef verison (pick the latest), you will be provided
a download link to the Omnibus Chef Windows installer.  After downloading
Run the install, choosing the default options.

The Chef Client installer also installs Ruby 1.9.3 for its own use in
the directory `C:\opscode\chef\embedded`.  You can also use this copy of Ruby
for your own cookbook development.

WARNING: Don't try to mix and match the Chef Client's Ruby 1.9.3 together
with a RVM/rbenv Ruby development setup.  Choose one or the other.  If your
Ruby needs go beyond Chef and writing Chef cookbooks, set up a "real"
RVM/rbenv Ruby environment.

Now that you have a Ruby 1.9.3 environment via the Chef Client install, you
can install any extra gem dependencies needed for writing Chef cookbooks by
using the `c:\opscode\chef\embedded\bin install` command.  For example,
here's how to install Berkshelf, a popular cookbook authoring support tool:

    > c:\opscode\chef\embedded\bin\gem install berkshelf --no-ri --no-rdoc
    Fetching: i18n-0.6.1.gem (100%)
    Fetching: multi_json-1.7.7.gem (100%)
    Fetching: activesupport-3.2.13.gem (100%)
    ...
    Successfully installed safe_yaml-0.9.3
    Successfully installed test-kitchen-1.0.0.alpha.7
    Successfully installed berkshelf-2.0.3
    43 gems installed

Add `c:\opscode\chef\embedded\bin` to your PATH environment variable:
![Environments Control Panel](/images/opscodechefpathwin.png)

Restart your Command Prompt to pick up the new environment variable setting,
and then you can run Berkshelf:
    
    > berks -v
    Berkshelf (2.0.3)
