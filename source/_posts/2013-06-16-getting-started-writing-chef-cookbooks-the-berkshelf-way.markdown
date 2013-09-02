---
layout: post
title: "Getting started writing Chef cookbooks the Berkshelf Way, Part 1"
date: 2013-06-16 03:49
comments: true
categories: chef
---
* list element with functor item
{:toc}

_Updated September 1st, 2013_

* _Corrected link to Jamie Winsor's Berkshelf slides_
* _Bumped berkshelf from version 2.0.8 to 2.0.9_
* _Bumped Test Kitchen version from 1.0.0.beta.2 to 1.0.0.beta.3_
* _Bumped apache2 cookbook reference from 1.6.x to 1.7.x_
* _Per Herr Flupke, tried to make the difference between files and templates more clear_

_Updated August 7th, 2013_

* _Bumped vagrant from version 1.2.4 to 1.2.7_
* _Bumped berkshelf from version 2.0.7 to 2.0.8_
* _Berkshelf 2.0.8 currently needs a version constraint for test-kitchen - added_

_Updated July 22nd, 2013_

* _Bumped vagrant from version 1.2.3 to 1.2.4_
* _Bumped berkshelf from version 2.0.6 to 2.0.7_
* _Referenced Sean O'Meara's & Charles Johnson's latest myface example app_

_Updated July 9th, 2013_

* _Bumped vagrant from version 1.2.2 to 1.2.3_
* _Bumped berkshelf from version 2.0.5 to 2.0.6_
* _Bumped VirtualBox from version 4.2.12 to 4.2.16_
* _Berkshelf 2.0.6 no longer needs the version constraint for test-kitchen - removed_

Jamie Winsor hasn't yet updated his [guide to authoring cookbooks the Berkshelf way](http://vialstudios.com/guide-authoring-cookbooks.html)
to match [recent changes related to Vagrant 1.x](https://github.com/RiotGames/berkshelf/issues/416) and [Chef 11](http://www.opscode.com/blog/2013/03/11/chef-11-server-up-and-running/)
This post is an attempt to update these instructions, walking through Jamie's
and Sean O'Meara's example app - [MyFace](https://github.com/someara/myface-cookbook).
For more information on [Berkshelf](http://berkshelf.com/), check out his recent
[talk](http://www.youtube.com/watch?v=hYt0E84kYUI)
and [slides](http://www.slideshare.net/resetexistence/the-berkshelf-way-21787019?from_search=1)
from ChefConf 2013.

NOTE: The source code examples covered in this article can be found on
GitHub: <https://github.com/misheska/myface>

Getting Started
===============
You can write Chef Cookbooks with Berkshelf on Mac OS X, Linux or Windows.
To set up your cookbook-writing environment, make sure you have the following
installed:

* [Install VirtualBox 4.2.x (or higher)](http://virtualbox.org)
NOTE: Avoid VirtualBox 4.2.14, [it breaks vagrant](https://twitter.com/mitchellh/status/348886504728305664).  VirtualBox 4.2.16 was tested for
this post.

* [Install Vagrant 1.2.1 (or higher)](http://vagrantup.com).  Vagrant 1.2.7 was tested for this post.

* Install Ruby 1.9.x via [Chef Omnibus Installer Ruby](http://misheska.com/blog/2013/06/16/use-opscode-chef-omnibus-ruby-for-writing-cookbooks/), [rvm](http://misheska.com/blog/2013/06/16/using-rvm-to-manage-multiple-versions-of-ruby/) or [rbenv](http://misheska.com/blog/2013/06/15/using-rbenv-to-manage-multiple-versions-of-ruby/)

* Install Berkshelf

```
$ gem install berkshelf --no-ri --no-rdoc
Fetching: berkshelf-2.0.9.gem (100%)
Successfully installed berkshelf-2.0.9
1 gem installed

$ berks -v
Berkshelf (2.0.9)

Copyright 2012-2013 Riot Games

    Jamie Winsor (<jamie@vialstudios.com>)
    Josiah Kiehl (<jkiehl@riotgames.com>)
    Michael Ivey (<michael.ivey@riotgames.com>)
    Justin Campbell (<justin.campbell@riotgames.com>)
    Seth Vargo (<sethvargo@gmail.com>)
```

* Install the vagrant-berkshelf Plugin (1.3.3 or higher)

```
$ vagrant plugin install vagrant-berkshelf
Installing the 'vagrant-berkshelf' plugin. This can take a few minutes...
Installed the plugin 'vagrant-berkshelf (1.3.3)'!
```

* Install the vagrant-omnibus plugin (1.1.0 or higher)

```
$ vagrant plugin install vagrant-omnibus
Installing the 'vagrant-omnibus' plugin.  This can take a few minutes...
Installed the plugin 'vagrant-omnibus (1.1.0)'!
```

Upgrade from Berkshelf 1.x
--------------------------
NOTE: If you had a previous 1.x version of the berkshelf plugin installed,
when it was named `berkshelf-vagrant`, which you can verify by running
the following command:

    $ vagrant plugin list
    berkshelf-vagrant (1.1.3)

Make sure you fully uninstall the old `berkshelf-vagrant` plugin before
installing the new `vagrant-berkshelf` plugin, as vagrant will get confused
by the name change:

    $ vagrant plugin uninstall berkshelf-vagrant
    Uninstalling the 'berkshelf-vagrant' plugin...
    $ vagrant plugin install vagrant-berkshelf
    Installing the 'vagrant-berkshelf' plugin.  This can take a few minutes...

Create the MyFace Application Cookbook
======================================
Key to the Berkshelf way is the use of the Application Cookbook Pattern.  An
application cookbook contains the list of recipes needed to build your
application or service.  As an example, this blog post will walk you through
the creation of an example service - MyFace - the next killer social web app.

First create a new cookbook for the MyFace application using the
`berks cookbook myface` command:

    $ berks cookbook myface
          create  myface/files/default
          create  myface/templates/default
          create  myface/attributes
          create  myface/definitions
          create  myface/libraries
          create  myface/providers
          create  myface/recipes
          create  myface/resources
          create  myface/recipes/default.rb
          create  myface/metadata.rb
          create  myface/LICENSE
          create  myface/README.md
          create  myface/Berksfile
          create  myface/Thorfile
          create  myface/chefignore
          create  myface/.gitignore
             run  git init from "./myface"
          create  myface/Gemfile
          create  .kitchen.yml
          append  Thorfile
          create  test/integration/default
          append  .gitignore
          append  .gitignore
          append  Gemfile
          append  Gemfile
    You must run `bundle install' to fetch any new gems.
          create  myface/Vagrantfile

Before running `bundle install` edit the `Gemfile` and add a version constraint
for the `test-kitchen` gem so that [Bundler](http://bundler.io/) can resolve
all the dependencies properly:

{% codeblock myface/Gemfile lang:ruby %}
source 'https://rubygems.org'

gem 'berkshelf'
gem 'test-kitchen', '~> 1.0.0.beta.3', :group => :integration
gem 'kitchen-vagrant', :group => :integration
{% endcodeblock %}

As of this writing, if you do not add the version constraint for `test-kitchen`
you will get this mysterious error when you run `bundle install`:

    Bundler could not find compatible versions for gem "test-kitchen":
      In Gemfile:
        kitchen-vagrant (>= 0) ruby depends on
          test-kitchen (~> 1.0.0.alpha.0) ruby

        test-kitchen (0.5.0)

The version constraint addresses this issue.

Run `bundle install` in the newly created cookbook directory to install the
necessary Gem dependencies:

    $ cd myface
    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.5)
    Using multi_json (1.7.9)
    Using activesupport (3.2.14)
    . . .
    Using berkshelf (2.0.9)
    Using coderay (1.0.9)
    Using mixlib-shellout (1.2.0)
    Using net-scp (1.1.2)
    Using method_source (0.8.2)
    Using slop (3.4.6)
    Using pry (0.9.12.2)
    Using safe_yaml (0.9.5)
    Using test-kitchen (1.0.0.beta.3)
    Using kitchen-vagrant (0.11.1)
    Using bundler (1.3.5)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.


Prepare a virtual machine for testing
=====================================
It's a good idea to develop your cookbook incrementally, testing 
in short iterations.  Berkshelf integrates with Vagrant to deploy
your cookbook changes to a virtual machine for testing.

Ensure that the `vagrant-omnibus` plugin is installed correctly.

    $ vagrant plugin list
    ...
    vagrant-omnibus (1.1.0)
    ...

The `vagrant-omnibus` plugin hooks into Vagrant and allows you to specify
the version of the Chef Omnibus package you want installed using the
`omnibus.chef_version` key

Edit the Vagrantfile generated by the `berks cookbook` command to use
a VirtualBox template that does not have a version of Chef provisioned.
Then, specify that you want your image to always use the latest version
of Chef.  (By default Berkshelf points to an image with an older version of CentOS and Chef 11.2.0, which is also old).  Your Vagrantfile should look like this:

{% codeblock myface/Vagrantfile lang:ruby %}
Vagrant.configure("2") do |config|
  config.vm.hostname = "myface-berkshelf"
  config.vm.box = "misheska-centos64"
  config.vm.box_url = "https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-centos64.box"
  config.omnibus.chef_version = :latest
  config.vm.network :private_network, ip: "33.33.33.10"
  config.ssh.max_tries = 40
  config.ssh.timeout = 120
  config.berkshelf.enabled = true

  config.vm.provision :chef_solo do |chef|
    chef.json = {
      :mysql => {
        :server_root_password => 'rootpass',
        :server_debian_password => 'debpass',
        :server_repl_password => 'replpass'
      }
    }
    chef.run_list = [
      "recipe[myface::default]"
    ]
  end
end
{% endcodeblock %}

Run `vagrant up` to start up the virtual machine and to test the stub MyFace
cookbook you just created:

    $ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    [default] Box 'misheska-centos64' was not found. Fetching box from specified URL for
    the provider 'virtualbox'. Note that if the URL does not have
    a box for this provider, you should interrupt Vagrant now and add
    the box yourself. Otherwise Vagrant will attempt to download the
    full box prior to discovering this error.
    Downloading or copying the box...
    Extracting box...te: 2487k/s, Estimated time remaining: --:--:--)
    Successfully added box 'misheska-centos64' with provider 'virtualbox'!
    [default] Importing base box 'misheska-centos64'...
    [default] Matching MAC address for NAT networking...
    [default] Setting the name of the VM...
    [default] Clearing any previously set forwarded ports...
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Setting hostname...
    [default] Configuring and enabling network interfaces...
    [default] Mounting shared folders...
    [default] -- /vagrant
    [default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Installing Chef 11.6.0 Omnibus package...
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T20:41:29-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T20:41:29-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T20:41:30-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T20:41:30-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T20:41:30-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T20:41:30-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T20:41:30-07:00] INFO: Running start handlers
    [2013-08-07T20:41:30-07:00] INFO: Start handlers complete.
    [2013-08-07T20:41:30-07:00] INFO: Chef Run complete in 0.025063914 seconds
    [2013-08-07T20:41:30-07:00] INFO: Running report handlers
    [2013-08-07T20:41:30-07:00] INFO: Report handlers complete

If all goes well, you should see `Chef Run complete` with no errors.

NOTE: The basebox URL comes from my current collection of baseboxes.  The
following link points to a README file which provides links to all the
vagrant baseboxes I use (which I normally update frequently):
<https://github.com/misheska/basebox-packer>

If you would ever like to delete your test virtual machine and start over,
you can destroy your test virtual machine with the `vagrant destroy` command:

    $ vagrant destroy
    Are you sure you want to destroy the 'default' VM? [y/N] y
    [default] Forcing shutdown of VM...
    [default] Destroying VM and associated drives...
    [Berkshelf] Cleaning Vagrant's berkshelf

Run `vagrant up` to recreate the test virtual machine.

__NOTE:__ If you just ran `vagrant destroy` make sure you run `vagrant up`
before proceeding to the next section.

Iteration #1: Create an application user
========================================
For our first short iteration, let's create a `myface` user under which
we'll run our application.  One best practice is to avoid running
applications as root and create a user/group under which the application runs
instead who has just enough rights that the app needs.

Edit `myface/recipes/default.rb` defining a new [Group Resource](http://docs.opscode.com/resource_group.html)
and [User Resource](http://docs.opscode.com/resource_user.html) for myface,
so it looks like the following:

{% codeblock myface/recipes/default.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group "myface"

user "myface" do
  group "myface"
  system true
  shell "/bin/bash"
end
{% endcodeblock %}

Save `recipes/default.rb` and re-run `vagrant provision` to create the
myface user on your test virtual machine:

    $ vagrant provision
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T20:46:33-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T20:46:33-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T20:46:34-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T20:46:34-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T20:46:34-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T20:46:34-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T20:46:34-07:00] INFO: Running start handlers
    [2013-08-07T20:46:34-07:00] INFO: Start handlers complete.
    [2013-08-07T20:46:34-07:00] INFO: group[myface] created
    [2013-08-07T20:46:34-07:00] INFO: user[myface] created
    [2013-08-07T20:46:34-07:00] INFO: Chef Run complete in 0.185753077 seconds
    [2013-08-07T20:46:34-07:00] INFO: Running report handlers
    [2013-08-07T20:46:34-07:00] INFO: Report handlers complete

You should expect to see the Chef run complete with no errors.  Notice
that it also creates `group[myface]` and `user[myface]`.

Testing Iteration #1
--------------------

Verify that Chef actually created the myface user on our test virtual
machine by running the following:

    $ vagrant ssh -c "getent passwd myface"
    myface:x:497:503::/home/myface:/bin/bash

We use `vagrant ssh -c` to run a command on our test virtual machine.  The
`getent` command can be used to query all user databases.  In this
case we're looking for `myface`, and it exists!

Because we are using well-defined resources that are completely
[idempotent](http://en.wikipedia.org/wiki/Idempotence), you should notice
that if you run `vagrant provision` again, the Chef run executes more quickly
and it does not try to re-create the user/group it already created.

    $ vagrant provision
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T20:48:27-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T20:48:27-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T20:48:28-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T20:48:28-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T20:48:28-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T20:48:28-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T20:48:28-07:00] INFO: Running start handlers
    [2013-08-07T20:48:28-07:00] INFO: Start handlers complete.
    [2013-08-07T20:48:28-07:00] INFO: Chef Run complete in 0.027612724 seconds
    [2013-08-07T20:48:28-07:00] INFO: Running report handlers
    [2013-08-07T20:48:28-07:00] INFO: Report handlers complete

Iteration #2 - Refactor to attributes
=====================================
What if at some point you wanted to change the name of the `myface` user/group
you just created to something else?  At the moment, you would need to edit
`myface/recipes/default.rb` in three places.

Let's create a new file called `myface/attributes/default.rb` which
initializes Chef [attributes](http://docs.opscode.com/essentials_cookbook_attribute_files.html)
defining the user name and group name under which our application will run so
that you [don't repeat yourself](http://en.wikipedia.org/wiki/Don't_repeat_yourself).

{% codeblock myface/attributes/default.rb lang:ruby %}
default[:myface][:user] = "myface"
default[:myface][:group] = "myface"
{% endcodeblock %}

In Chef, attributes are a hash of a hash used to override the default settings
on a node.  The first hash is the cookbook name - in our
case we've named our cookbook `:myface`. The second hash is the name of
our attribute - in this case, we're defining two new attributes: `:user` and
`:group`.

`default` implies the use of the [node object](http://docs.opscode.com/chef/essentials_node_object.html)
`node.default` and is a Chef attribute file shorthand.  The following are
equivalent definitions to the ones above:

    node.default[:myface][:user] = "myface"
    node.default[:myface][:group] = "myface"

Also note the use of symbols instead of strings.  It is [strongly recommended
that you use symbols instead of strings](http://www.robertsosinski.com/2009/01/11/the-difference-between-ruby-symbols-and-strings/)
for hash indexes. 

Now that you've created your attribute definitions, edit
`myface/recipes/default.rb` and replace all references to the "myface" user name
with `node[:myface][:user]` and all references to the "myface" group with
`node[:myface][:group]`.  `myface/recipes/default.rb` should now look like
this:

{% codeblock myface/recipes/default.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end
{% endcodeblock %}

Re-provision with `vagrant provision` to see how the refactor went:

    $ vagrant provision

As long as you didn't create any syntax errors in your refactoring file edits,
there should be no net change on the virtual machine test node (as you've only
just moved some strings into a node attribute).  

Testing Iteration #2
--------------------

Running  `getent` on the test virtual machine should also produce the same
result as when you validated Iteration #1:

    $ vagrant ssh -c "getent passwd myface"
    myface:x:497:503::/home/myface:/bin/bash

Iteration #3 - Install the Apache2 Web Server
============================================
Our hot new social networking application, myface, is a web app, so we need
to install a web server.  Let's install the Apache2 web server.

Modify `myface/recipes/default.rb` to include the apache2 cookbook's default
recipe:

    include_recipe "apache2"

The resultant `myface/recipes/default.rb` file should look like so:

{% codeblock myface/recipes/default.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"
{% endcodeblock %}

Since you are loading Apache2 from another cookbook, you need to configure the
dependency in your metadata.  Edit `myface/metadata.rb` and add the `apache2`
dependency at the bottom:

    depends "apache2", "~> 1.7.0"

This tells Chef that the myface cookbook depends on the apache2 cookbook.
We've also specified a version constraint using the optimistic operator
`~>` to tell our Chef that we want the latest version of the apache2 cookbook
that is greater than 1.7.0 but *not* 1.8.0 or higher.

It is recommended that Chef cookbooks follow a
[Semantic Versioning](http://semver.org/) scheme.  So if you write your
cookbook to use the latest apache2 1.7.x cookbook, if the apache2 cookbook is
ever bumped to a 1.8.x version (or 2.x), it has some public API functionality
that has at least been deprecated.  So you'll want to review the changes and
test before automatically using an apache2 cookbook version 1.8.x or higher.
However, automatic use of any new 1.7.x is perfectly fine, because no
only backwards-compatible bug fixes has been introduced.  Semantic Versioning
guarantees there are no changes in the public APIs.

Your `myface/metadata.rb` file should look like this:

{% codeblock myface/metadata.rb lang:ruby %}
name             "myface"
maintainer       "YOUR_NAME"
maintainer_email "YOUR_EMAIL"
license          "All rights reserved"
description      "Installs/Configures myface"
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          "0.1.0"

depends "apache2", "~> 1.7.0"
{% endcodeblock %}

Now when you re-run `vagrant provision` it will install apache2 on your
test virtual machine:

    $ vagrant provision
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (0.1.0)
    [Berkshelf] Installing apache2 (1.7.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T20:53:08-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T20:53:08-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T20:53:08-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T20:53:08-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T20:53:08-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T20:53:08-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T20:53:08-07:00] INFO: Running start handlers
    [2013-08-07T20:53:08-07:00] INFO: Start handlers complete.
    ...
    [2013-08-07T20:54:13-07:00] INFO: service[apache2] started
    [2013-08-07T20:54:13-07:00] INFO: template[/etc/httpd/mods-available/deflate.conf] sending restart action to service[apache2] (delayed)
    [2013-08-07T20:54:14-07:00] INFO: service[apache2] restarted
    [2013-08-07T20:54:14-07:00] INFO: Chef Run complete in 66.177345199 seconds
    [2013-08-07T20:54:14-07:00] INFO: Running report handlers
    [2013-08-07T20:54:14-07:00] INFO: Report handlers complete

Testing Iteration #3
--------------------

You can verify that the apache2 `httpd` service is running on your berkshelf
virtual machine with the following command:

    $ vagrant ssh -c "sudo /sbin/service httpd status"
    httpd (pid  4831) is running.

Since this is a web server, so you can also check it out in your favorite web
browser.  The host-only private network address for the virtual machine
that Berkshelf created is in the `Vagrantfile`.  Display the IP address with
the following command:

    $ grep ip: Vagrantfile
    config.vm.network :private_network, ip: "33.33.33.10"

Check it out with your favorite web browser:

<http://33.33.33.10>

While you will get a `404 Not Found` error because we haven't added any
content to our web site yet, the important part is that `Apache Server
at 33.33.33.10 Port 80` sent the response:

![Apache Server Response](/images/apachewebserver.png)

Wait a second, though.  You never downloaded the `apache2` cookbook!
That's the magic of the Berkshelf Vagrant plugin you installed earlier.  The
Berkshelf Vagrant plugin will make sure that any changes you make to your
cookbook and all of your cookbook's dependencies are made available to your
virtual machine.  Berkshelf automatically loads all your cookbook dependencies
much like Bundler automatically loads all your gem dependencies.

Where does the Berkshelf put the cookbooks it downloads?  You can find them
in `~/.berkshelf/default/cookbooks`

    Users/misheska/.berkshelf/default/cookbooks
    └── apache2-1.7.0
        ├── attributes
        ├── definitions
        ├── files
        │   └── default
        │       └── tests
        │           └── minitest
        │               └── support
        ├── recipes
        └── templates
            └── default
                └── mods

`~/.berkshelf` is just the default location where Berkshelf stores data
on your local disk.  This location can be altered by setting the environment
variable `BERKSHELF_PATH`.

Iteration #4 - Add Content
==========================
Let's add some content to make that 404 go away.  Edit
`myface/recipes/default.rb` as follows:

{% codeblock myface/recipes/default.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"

# disable default site
apache_site "000-default" do
  enable false
end

# create apache config
template "#{node[:apache][:dir]}/sites-available/myface.conf" do
  source "apache2.conf.erb"
  notifies :restart, 'service[apache2]'
end

# create document root
directory "/srv/apache/myface" do
  action :create
  recursive true
end

# write site
cookbook_file "/srv/apache/myface/index.html" do
  mode "0644" # forget me to create debugging exercise
end

# enable myface
apache_site "myface.conf" do
  enable true
end
{% endcodeblock %}

If you're familiar with Chef and configuring a web app via apache2, nothing
here should be too surprising.  But if not, spend some time reading up on
the resource references at <http://docs.opscode.com>

With Chef, you can create config files from templates using
[ERB](http://ruby-doc.org/stdlib-1.9.3/libdoc/erb/rdoc/ERB.html), a
Ruby templating system.  When the template `.erb` file is processed by Chef,
it will replace any token bracketed by a `<%=` `%>` tag with the value
of the Ruby expression inside the token (like `node[:hostname]`).  Chef expects
ERB template files to be in the `templates` subirectory of a cookbook.
A subdirectory level underneath `templates` is used to specify
platorm-specific files.  Files in the `templates/default` subdirectory are
used with every platform.

Create a new template file called
`myface/templates/default/apache2.conf.erb` which will become the
file `.../sites-available/myface.conf` on our test virtual machine
(refer to `myface/recipes/default.rb` above):

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node[:hostname] %>

Alias / /srv/apache/myface/

<Directory /srv/apache/myface>
	Options FollowSymLinks +Indexes
	Allow from All
</Directory>
{% endcodeblock %}

Create a file to contain our web site content as
`myface/files/default/index.html`.  `index.html` is a static file that does
not take advantage of any ERB templating.  Chef looks for static files in
the `files` subtree by default.

{% codeblock myface/files/default/index.html lang:ruby %}
Welcome to MyFace!
{% endcodeblock %}

After you have created these three files, run `vagrant provision` to deploy
your changes:

    $ vagrant provision

Testing Iteration #4
--------------------

If the Chef run completed successfully, if you point your web browser at your
myface web site again:

<http://33.33.33.10>

You'll see some lovely content!

![Welcome to MyFace](/images/welcometomyface.png)

Iteration #5 - Refactoring webserver
====================================
`myface/recipes/default.rb` is getting rather large and we've got a lot more
to add to our cookbook.  Let's go through another refactoring pass.

Let's move all the webserver-related resources to their own file
`myface/recipes/webserver.rb`.  Rename `myface/recipes/default.rb` to
`myface/recipes/webserver.rb`.

Now `myface/recipes/webserver.rb` should look like this: 

{% codeblock myface/recipes/webserver.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: webserver
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"

# disable default site
apache_site "000-default" do
  enable false
end

# create apache config
template "#{node[:apache][:dir]}/sites-available/myface.conf" do
  source "apache2.conf.erb"
  notifies :restart, 'service[apache2]'
end

# create document root
directory "/srv/apache/myface" do
  action :create
  recursive true
end

# write site
cookbook_file "/srv/apache/myface/index.html" do
  mode "0644"
end

# enable myface
apache_site "myface.conf" do
  enable true
end
{% endcodeblock %}

Create a new `myface/recipes/default.rb` file which references `webserver.rb`.

{% codeblock myface/recipes/default.rb lang:ruby %}

# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

include_recipe "myface::webserver"
{% endcodeblock %}

Converge your node again to make sure there are no syntax errors:

    $ vagrant provision

Let's eliminate some more of the duplication that crept in while we were
working on things.  Edit `myface/attributes/default.rb`

{% codeblock myface/attributes/default.rb lang:ruby %}
default[:myface][:user] = "myface"
default[:myface][:group] = "myface"
default[:myface][:name] = "myface"
default[:myface][:config] = "myface.conf"
# Trailing slash on document_root is important
default[:myface][:document_root] = "/srv/apache/myface/"
{% endcodeblock %}

NOTE: With Chef 11, it is now possible to nest attributes, like so:

    node.default[:app][:name] = "my_app"
    node.default[:app][:document_root] = "/srv/apache/#{node[:app][:name]}"

This approach is overkill for MyFace (and is frankly overkill for most
Chef recipes).  Even though nesting is an option now with Chef 11, you should
try to keep your attribute files as simple and straightforward to follow as
possible.

In `myface/recipes/webserver.rb` replace the corresponding hardcoded references
to attribute references: 

{% codeblock myface/recipes/webserver.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: webserver
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"

# disable default site
apache_site "000-default" do
  enable false
end

# create apache config
template "#{node[:apache][:dir]}/sites-available/#{node[:myface][:config]}" do
  source "apache2.conf.erb"
  notifies :restart, 'service[apache2]'
end

# create document root
directory "#{node[:myface][:document_root]}" do
  action :create
  recursive true
end

# write site
cookbook_file "#{node[:myface][:document_root]}index.html" do
  mode "0644"
end

# enable myface
apache_site "#{node[:myface][:config]}" do
  enable true
end
{% endcodeblock %}

Converge your node to make sure there are no syntax errors:

    $ vagrant provision

Add tokens to the `templates/default/apache2.conf.erb` ERB template
file so they will be replaced with the value of the
`node[:myface][:document_root]` attribute during the Chef run.

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node[:hostname] %>

Alias / <%= node[:myface][:document_root] %>

<Directory <%= node[:myface][:document_root] %>>
    Options FollowSymLinks +Indexes
    Allow from All
</Directory>
{% endcodeblock %}

Converge your node one last time to make sure there are no syntax errors:

    $ vagrant provision

Testing Iteration #5
--------------------

Visiting <http://33.33.33.10> should produce the same result as before as you
have made no net changes, just shuffled things around a bit.

Iteration #6 - Version Bump and Production Deploy
=================================================
Now that we have tested our cookbook locally and everything seems to work,
we're ready to finalize the cookbook and deploy it to production.

Version Bump to 1.0.0
---------------------

First you need to "bump" the cookbook version in the `metadata.rb` file to
its final version 1.0.0.  As mentioned in Iteration #3, cookbooks (even the
ones you write), should follow the
[Semantic Versioning scheme](http://semver.org/).  Since this is the first
public version of our cookbook, it's version 1.0.0.

`myface/metadata.rb` should look like this:

{% codeblock myface/metadata.rb lang:ruby %}
name             "myface"
maintainer       "Mischa Taylor"
maintainer_email "mischa@misheska.com"
license          "Apache 2.0"
description      "Installs/Configures myface"
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          "1.0.0"

depends "apache2", "~> 1.7.0"
{% endcodeblock %}

Configure Berkshelf
-------------------

In order to deploy to your production server (instead of just locally with
vagrant), you'll need to add a section to your Berkshelf config file with
your production Chef settings.  The config file is located in
`$HOME/.berkshelf/config.json`.

For example, I'm using hosted Chef to manage my servers, so my
Chef settings are as follows:

    Chef Server API URL endpoint: https://api.opscode.com/organizations/misheska
    Chef API validation client name: misheska-validator
    Chef API validation key path: /Users/misheska/.chef/misheska-validator.pem
    Chef API client key: /Users/misheska/.chef/misheska.pem
    Chef API node name: misheska

So here's what my `$HOME/.berkshelf/config.json` file looks like:

    {
      "chef":{
        "chef_server_url": "https://api.opscode.com/organizations/misheska",
        "validation_client_name": "misheska-validator",
        "validation_key_path": "/Users/mischa/.chef/misheska-validator.pem",
        "client_key": "/Users/mischa/.chef/misheska.pem",
        "node_name":"misheska"
      },
      "vagrant":{
        "vm":{
          "box": "misheska-centos64",
          "box_url":"https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-centos64.box",
          "forward_port": {
          },
          "network":{
            "bridged": false,
            "hostonly": "33.33.33.10"
          },
          "provision": "chef_solo"
        }
      },
      "ssl": {
        "verify":false
      }
    }

I assume you have your own production Chef setup either running 
[Hosted Chef](http://www.opscode.com/hosted-chef/),
[Private Chef](http://www.opscode.com/private-chef/), or
[Open Source Chef Server](http://docs.opscode.com/chef/manage_server_open_source.html).  If you need more help getting your .pem file values, refer to
the [QuickStart Guide on LearnChef](https://learnchef.opscode.com/quickstart/chef-repo/).

Upload cookbooks
----------------

Edit your `$HOME/.berkshelf/config.json` file similarly with your .pem file
values.  Then run `berks upload` to upload your cookbooks to the chef server:

    $ berks upload
    Using myface (1.0.0)
    Using apache2 (1.7.0)
    Uploading myface (1.0.0) to: 'https://api.opscode.com:443/organizations/misheska'
    Uploading apache2 (1.7.0) to: 'https://api.opscode.com:443/organizations/misheska'

Similarly to when you ran `vagrant up`, Berkshelf automatically uploaded all
the cookbook dependencies.

Converge node
-------------

Add the default `myface` cookbook recipe to your node's run list.  For example,
I used the following command to add `myface` to mine:

    $ knife node run_list add mischa-ubuntu 'recipe[myface]'
    mischa-ubuntu:
      run_list: recipe[myface]

Converge the node by running `chef-client` (if you don't already have it
setup to run chef-client automatically).  For example, here's the command
I used to run `chef-client` on my production node:

    $ knife ssh name:mischa-ubuntu "sudo chef-client" -x misheska
    Starting Chef Client, version 11.6.0
    resolving cookbooks for run list: ["myface"]
    Synchronizing Cookbooks:
      - myface
      - apache2
    ...
    Chef Client finished, 20 resources updated 

Testing Iteration #6
--------------------

Browsing your production node's URL should produce the same result as when
you tested Iteration #4.  For example, I visited <http://mischa-ubuntu> 

More to Come!
=============
This is just part one of a multi-part series.  So far you've gone through
several short iteration loops as you evolve the myface cookbook web
application.  In [Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/),
we'll wire up a database to the myface application and explore the use of
the `mysql` and `database` cookbooks.

If you want to see the full source for myface, check out the following
GitHub link: <https://github.com/misheska/myface>
