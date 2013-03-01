---
layout: post
title: "Try out Sensu Monitoring using Virtual Box, Vagrant and Chef"
date: 2013-02-28 22:54
comments: true
categories: [ruby, virtual box, vagrant, chef]
---
I've been using Sensu Monitoring in production for about three to four months
now.  It's a nice, lightweight monitoring framework, designed with the cloud
in mind and for use with modern configuration management frameworks like
Chef and Puppet.  For more information on Sensu, check out the article
[Why Sonian created Sensu (by Sean Porter)](https://github.com/sensu/sensu/wiki)
and the associated articles and links on the [Sensu Wiki](https://github.com/sensu/sensu/wiki)

In this article, I'm going to present a quick overview on how to test and
evaluate Sensu using the [Sensu Chef Cookbook](https://github.com/sensu/sensu-chef)
Through the magic of Oracle VirtualBox and Vagrant, combined with Chef, you
can quickly deploy Sensu to a local virtual machine instance, and kick the
tires on Sensu to evaluate whether or not it is a good monitoring solution
for you.  These instructions assume you are using either Mac OS X or Linux.  As
time permits, I will eventually update this article with instructions for
Windows as well.

Install VirtualBox
==================
VirtualBox is an open source virtualization platform, similar to VMWare
Fusion/Workstation that runs on Mac OS X, Linux and Windows (and a few more
platforms).  While I personally prefer VMWare Fusion to VirtualBox (VirtualBox
can be a bit flakey at times), a lot of automation around VirtualBox has been
developed within the Chef community, which impossible to ignore.  Fortunately
on both Mac OS X and Linux, VirtualBox can peacefully coexist with VMWare
Fusion/Workstation (NOTE: If you use KVM virtualization on Linux, VirtualBox
can also coexist, but you need to be careful not to run VirtualBox and KVM
images simultaneously).

Just download and run the VirtualBox installer from the [VirtualBox download
page](https://www.virtualbox.org/wiki/Downloads).  Just download and install
the latest 4.2.x version of VirtualBox.  Make sure you save the VirtualBox
installer as it comes with an uninstall tool, should you wish to remove
VirtualBox at some point in the future.

Install Vagrant
===============
After installing VirtualBox, next install Vagrant.  Vagrant is an automation
framework for VirtualBox.  Grab the latest Vagrant installer for your OS from
the [Vagrant Downloads page](http://downloads.vagrantup.com/) and run install.
On Mac OS X, the Vagrant install will automatically add the Vagrant binaries
to your PATH, on Linux, you will need to manually add
<code>/opt/vagrant/bin</code> per the [Getting Started with Vagrant docs](http://docs.vagrantup.com/v1/docs/getting-started/index.html).

Download the sensu-chef cookbook
================================
Grab the latest version of the sensu-chef cookbook from GitHub by running
the following command:
```
git clone https://github.com/sensu/sensu-chef.git
```
Install Ruby & RubyGems
=======================
The sensu-chef cookbook requires Ruby & RubyGems.  I strongly recommend that
you use either RVM or Rbenv to make sure that you are using the latest version
of Ruby instead of whatever version of Ruby your system installs by default.
See my previous articles on [RVM](http://misheska.com/blog/2013/02/24/using-rvm-to-manage-multiple-versions-of-ruby/) or [Rbenv](http://misheska.com/blog/2013/02/24/using-rbenv-to-manage-multiple-versions-of-ruby/).  NOTE: If you don't know whether or not to decide between RVM or Rbenv,
go with RVM.

Install the Vbguest plugin
==========================
To make sure that the VirtualBox Guest Additions are always in sync with
the vagrant boxes you might download, make sure you install the
[Vbguest plugin](https://github.com/dotless-de/vagrant-vbguest) by
running the following command:
```
vagrant gem install vagrant-vbguest
```

After installing, the vbguest plugin will always check & install the correct
guest additions when you run <code>vagrant up</code>

For more information on why this is necessary, check out this awesome
post by Kevin van Zonneveld [Sync VirtualBox Guest Additions](http://kvz.io/blog/2013/01/16/vagrant-tip-keep-virtualbox-guest-additions-in-sync/)

Patch chef-sensu Vagrantfile
============================
As of this writing, the <code>Vagrantfile</code> included in
<code>sensu-chef/examples</code> will not set up the VM properly and the
Chef run will fail with the following error:
```
[default] Running chef-solo...
stdin: is not a tty
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: *** Chef 0.10.8 ***
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Setting the run_list to ["recipe[monitor::master]", "recipe[monitor::redis]", "recipe[monitor::rabbitmq]"] from JSON
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Run List is [recipe[monitor::master], recipe[monitor::redis], recipe[monitor::rabbitmq]]
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Run List expands to [monitor::master, monitor::redis, monitor::rabbitmq]
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Starting Chef Run for ubuntu-1204-i386
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Running start handlers
[Fri, 01 Mar 2013 08:28:06 +0000] INFO: Start handlers complete.
[Fri, 01 Mar 2013 08:28:07 +0000] ERROR: Running exception handlers
[Fri, 01 Mar 2013 08:28:07 +0000] ERROR: Exception handlers complete
[Fri, 01 Mar 2013 08:28:07 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
[Fri, 01 Mar 2013 08:28:07 +0000] FATAL: NoMethodError: undefined method `default_action' for #<Class:0x8decb20>
Chef never successfully completed! Any errors should be visible in the
output above. Please fix your recipes so that they properly complete.
```

I've created a revised <code>Vagrantfile</code> which fixes this issue.
Download this amended version and copy it in place of
<code>sensu-client/examples/Vagrantfile</code>

{% gist 5063291 Vagrantfile %}

Create the sensu-chef virtual machine
=====================================

Run the following commands to create the sensu-chef virtual machine:
```
cd sensu-chef/examples
gem install bundler
bundle install
librarian-chef install
vagrant up
```

If all goes well, the chef-solo run should have succeeded, and you should
be able to view the Sensu dashboard by going to the following URL with
the username <code>admin</code> and the password <code>secret</code>:
[http://localhost:8080](http://localhost:8080)

If this is successful, just run the following command to log in to your
newly-created virtual machine instance:
```
vagrant ssh
```

And refer to the [Sensu wiki](https://github.com/sensu/sensu/wiki) on how 
to experiment with various configuration options.
