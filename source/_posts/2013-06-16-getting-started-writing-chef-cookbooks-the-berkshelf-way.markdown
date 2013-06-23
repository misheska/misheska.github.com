---
layout: post
title: "Getting started writing Chef cookbooks the Berkshelf Way"
date: 2013-06-16 03:49
comments: true
categories: chef
---
* list element with functor item
{:toc}

Jamie Winsor hasn't yet updated his [guide to authoring cookbooks the Berkshelf way](http://vialstudios.com/guide-authoring-cookbooks.html)
to match [recent changes related to Vagrant 1.x](https://github.com/RiotGames/berkshelf/issues/416) and [Chef 11](http://www.opscode.com/blog/2013/03/11/chef-11-server-up-and-running/)
This post is an attempt to update these instructions, walking through his
and Sean O'Meara's example app - [MyFace](https://github.com/reset/myface-cookbook).
For more information on [Berkshelf](http://berkshelf.com/), check out his recent
[talk](http://www.youtube.com/watch?v=hYt0E84kYUI)
and [slides](http://www.slideshare.net/resetexistence/the-berkshelf-way)
from ChefConf 2013.

NOTE: The source code examples covered in this article can be found on
Github: <https://github.com/misheska/myface>

Getting Started
===============
You can write Chef Cookbooks with Berkshelf on Mac OS X, Linux or Windows.
To set up your cookbook-writing environment, make sure you have the following
installed:

* [Install VirtualBox 4.2.x (or higher)](http://virtualbox.org)

* [Install Vagrant 1.2.1 (or higher)](http://vagrantup.com)

* Install Ruby 1.9.x via [Chef Omnibus Installer Ruby](http://misheska.com/blog/2013/06/16/use-opscode-chef-omnibus-ruby-for-writing-cookbooks/), [rvm](http://misheska.com/blog/2013/06/16/using-rvm-to-manage-multiple-versions-of-ruby/) or [rbenv](http://misheska.com/blog/2013/06/15/using-rbenv-to-manage-multiple-versions-of-ruby/)

* Install Berkshelf

```
$ gem install berkshelf --no-ri --no-rdoc
Fetching: berkshelf-2.0.5.gem (100%)
Successfully installed berkshelf-2.0.5
1 gem installed
$ berks -v
Berkshelf (2.0.5)
```

* Install the vagrant-berkshelf Plugin (1.3.2 or higher)

```
$ vagrant plugin install vagrant-berkshelf
Installing the 'vagrant-berkshelf' plugin. This can take a few minutes...
Installed the plugin 'vagrant-berkshelf (1.3.2)'!
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
`berks cookbook` command:

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

Before running `bundle install` edit the `Gemfile` and add a version
constraint for the `test-kitchen` gem, like so:

{% codeblock myface/Gemfile lang:ruby %}
source 'https://rubygems.org'

gem 'berkshelf'
gem 'test-kitchen', '~> 1.0.0.alpha', :group => :integration
gem 'kitchen-vagrant', :group => :integration
{% endcodeblock %}

Run `bundle install` in the newly created cookbook directory to install the
necessary Gem dependencies:

    $ cd myface
    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.1)
    Using multi_json (1.7.7)
    Using activesupport (3.2.13)
    . . .
    Using test-kitchen (1.0.0.alpha.7)
    Using kitchen-vagrant (0.10.0)
    Using bundler (1.3.5)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

If you get the following error, you forgot to add a version-constraint
for `test-kitchen`, per above:

    Bundler could not find compatible versions for gem "test-kitchen":
      In Gemfile:
        kitchen-vagrant (>= 0) ruby depends on
          test-kitchen (~> 1.0.0.alpha.0) ruby

        test-kitchen (0.5.0)

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
of Chef. (By default Berkshelf points to an image with an older version
of CentOS, Chef 11.2.0, which is also old).  Your Vagrantfile should look
like this:

{% codeblock myface/Vagrantfile lang:ruby %}
Vagrant.configure("2") do |config|
  config.vm.hostname = "myface-berkshelf"
  config.vm.box = "misheska-centos-6.4"
  config.vm.box_url = "https://www.dropbox.com/s/y42egyh9cqsge24/misheska-centos-6.4.box"
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
      "recipe[myface::defult]"
    ]
  end
end
{% endcodeblock %}

Run `vagrant up` to start up the virtual machine and to test the stub MyFace
cookbook you just created:

    $ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    [default] Box 'mishesksa-centos-6.4' was not found. Fetching box from specified URL for
    the provider 'virtualbox'. Note that if the URL does not have
    a box for this provider, you should interrupt Vagrant now and add
    the box yourself. Otherwise Vagrant will attempt to download the
    full box prior to discovering this error.
    Downloading or copying the box...
    Extracting box...te: 3449k/s, Estimated time remaining: 0:00:01)
    Successfully added box 'mishesksa-centos-6.4' with provider 'virtualbox'!
    [default] Importing base box 'mishesksa-centos-6.4'...
    [default] Matching MAC address for NAT networking...
    [default] Setting the name of the VM...
    [default] Clearing any previously set forwarded ports...
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/vagrant/berkshelf-20130616-17393-1q2qkbs'
    [Berkshelf] Using myface (0.1.0) at path: '/Users/misheska/xx/myface'
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Ensuring Chef is installed at requested version of 11.4.4.
    [default] Chef 11.4.4 Omnibus package is not installed...installing now.
    Downloading Chef 11.4.4 for el...
    Installing Chef 11.4.4
    warning: /tmp/tmp.Ro7kRMDB/chef-11.4.4.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
    Preparing...                ##################################################
    chef                        ##################################################
    Thank you for installing Chef!
    [default] Setting hostname...
    [default] Configuring and enabling network interfaces...
    [default] Mounting shared folders...
    [default] -- /vagrant
    [default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-06-16T10:28:25-07:00] INFO: *** Chef 11.4.4 ***
    [2013-06-16T10:28:25-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-06-16T10:28:25-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-06-16T10:28:25-07:00] INFO: Run List expands to [myface::default]
    [2013-06-16T10:28:25-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-06-16T10:28:25-07:00] INFO: Running start handlers
    [2013-06-16T10:28:25-07:00] INFO: Start handlers complete.
    [2013-06-16T10:28:25-07:00] INFO: Chef Run complete in 0.027690068 seconds
    [2013-06-16T10:28:25-07:00] INFO: Running report handlers
    [2013-06-16T10:28:25-07:00] INFO: Report handlers complete

If all goes well, you should see `Chef Run complete` with no errors.

NOTE: The basebox URL comes from my current collection of baseboxes.  The
following link points to a README file which provides links to all the
vagrant baseboxes I use (which I normally update frequently):
<https://github.com/misheska/basebox>

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

Save `receipes/default.rb` and re-run `vagrant provision` to create the
myface user on your test virtual machine:

    $ vagrant provision
    [default] Ensuring Chef is installed at requested version of 11.4.4.
    [default] Chef 11.4.4 Omnibus package is already installed...skipping installation.
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/vagrant/berkshelf-20130616-17393-1q2qkbs'
    [Berkshelf] Using myface (0.1.0) at path: '/Users/misheska/xx/myface'
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-06-16T10:36:32-07:00] INFO: *** Chef 11.4.4 ***
    [2013-06-16T10:36:32-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-06-16T10:36:32-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-06-16T10:36:32-07:00] INFO: Run List expands to [myface::default]
    [2013-06-16T10:36:32-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-06-16T10:36:32-07:00] INFO: Running start handlers
    [2013-06-16T10:36:32-07:00] INFO: Start handlers complete.
    [2013-06-16T10:36:32-07:00] INFO: Processing group[myface] action create (myface::default line 10)
    [2013-06-16T10:36:32-07:00] INFO: group[myface] created
    [2013-06-16T10:36:32-07:00] INFO: Processing user[myface] action create (myface::default line 12)
    [2013-06-16T10:36:32-07:00] INFO: user[myface] created
    [2013-06-16T10:36:32-07:00] INFO: Chef Run complete in 0.144073876 seconds
    [2013-06-16T10:36:32-07:00] INFO: Running report handlers
    [2013-06-16T10:36:32-07:00] INFO: Report handlers complete

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
    [default] Ensuring Chef is installed at requested version of 11.4.4.
    [default] Chef 11.4.4 Omnibus package is already installed...skipping installation.
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/vagrant/berkshelf-20130616-17393-1q2qkbs'
    [Berkshelf] Using myface (0.1.0) at path: '/Users/misheska/xx/myface'
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-06-16T10:39:11-07:00] INFO: *** Chef 11.4.4 ***
    [2013-06-16T10:39:12-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-06-16T10:39:12-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-06-16T10:39:12-07:00] INFO: Run List expands to [myface::default]
    [2013-06-16T10:39:12-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-06-16T10:39:12-07:00] INFO: Running start handlers
    [2013-06-16T10:39:12-07:00] INFO: Start handlers complete.
    [2013-06-16T10:39:12-07:00] INFO: Processing group[myface] action create (myface::default line 10)
    [2013-06-16T10:39:12-07:00] INFO: Processing user[myface] action create (myface::default line 12)
    [2013-06-16T10:39:12-07:00] INFO: Chef Run complete in 0.033772034 seconds
    [2013-06-16T10:39:12-07:00] INFO: Running report handlers
    [2013-06-16T10:39:12-07:00] INFO: Report handlers complete

Iteration #2 - Refactor to attributes
=====================================
What if at some point you wanted to change the name of the `myface` user/group
you just created to something else?  At the moment, you would need to edit
`myface/recipes/default.rb` in three places.

Let's create a new file called `myface/attributes/default.rb` which
initializes Chef [attributes](http://docs.opscode.com/essentials_cookbook_attribute_files.html)
defining the user name and group name under which our application will run so
that you [don't repeat yoursef](http://en.wikipedia.org/wiki/Don't_repeat_yourself).

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
    noce.default[:myface][:user] = "myface"

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

    depends "apache2", "~> 1.6.0"

This tells Chef that the myface cookbook depends on the apache2 cookbook.
We've also specified a version constraint using the optimistic operator
`~>` to tell our Chef that we want the latest version of the apache2 cookbook
that is greater than 1.6.0 but *not* 1.7.0 or higher.

It is recommended that Chef cookbooks follow a
[Semantic Versioning](http://semver.org/) scheme.  So if you write your
cookbook to use the latest apache2 1.6.x cookbook, if the apache2 cookbook is
ever bumped to a 1.7.x version (or 2.x), it has some public API functionality
that has at least been deprecated.  So you'll want to review the changes and
test before automatically using an apache2 cookbook version 1.7.x or higher.
However, automatic use of any new 1.6.x is perfectly fine, because no
only backwards-compatible bug fixes has been introduced.  Semantic Versioning
guarantees there are no changes in the public APIs.

Your `myface/metadata.rb` should look like this:

{% codeblock myface/attributes/default.rb lang:ruby %}
name             "myface"
maintainer       "YOUR_NAME"
maintainer_email "YOUR_EMAIL"
license          "All rights reserved"
description      "Installs/Configures myface"
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          "0.1.0"

depends "apache2", "~> 1.6.0"
{% endcodeblock %}

Now when you re-run `vagrant provision` it will install apache2 on your
test virtual machine:

    $ vagrant provision
    ...
    [2013-03-27T06:30:41+00:00] INFO: service[apache2] restarted
    [2013-03-27T06:30:41+00:00] INFO: execute[a2enmod deflate] sending restart action to service[apache2] (delayed)
    [2013-03-27T06:30:41+00:00] INFO: Processing service[apache2] action restart (apache2::default line 24)
    [2013-03-27T06:30:43+00:00] INFO: service[apache2] restarted
    [2013-03-27T06:30:43+00:00] INFO: Chef Run complete in 59.309557362 seconds
    [2013-03-27T06:30:43+00:00] INFO: Running report handlers
    [2013-03-27T06:30:43+00:00] INFO: Report handlers complete

Testing Iteration #3
--------------------

You can verify that the apache2 `httpd` service is running on your berkshelf
virtual machine with the following command:

    $ vagrant ssh -c "sudo /sbin/service httpd status"
    httpd (pid 5206) is running.

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
in `~/.berkshelf/cookbooks`

    Users/misheska/.berkshelf/cookbooks
    └── apache2-1.6.0
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
template "/srv/apache/myface/index.html" do
  source "index.html.erb"
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
Ruby templating system.  Create a new template file called
`myface/templates/default/apache2.conf.erb` which will become the
file `.../sites-available/myface.conf` on our test virtual machine
(refer to `myface/recipes/default.rb` above):

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node[:hostname] %>

Alias / /srv/apache/myface/

<Directory /srv/apache/myface >
	Options FollowSymLinks +Indexes
	Allow from All
</Directory>
{% endcodeblock %}

Also create our web site content as `myface/templates/default/index.html.erb`.
While it doesn't take advantage of ERB templating yet, it will in further
iterations.

{% codeblock myface/templates/default/index.html.erb lang:ruby %}
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
template "/srv/apache/myface/index.html" do
  source "index.html.erb"
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
default[:myface][:document_root] = "/srv/apache/myface"
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
template "#{node[:myface][:document_root]}/index.html" do
  source "index.html.erb"
  mode "0644"
end

# enable myface
apache_site "#{node[:myface][:config]}" do
  enable true
end
{% endcodeblock %}

Converge your node one last time to make sure there are no syntax errors:

    $ vagrant provision


Testing Iteration #5
--------------------

Visiting <http://33.33.33.10> should produce the same result as before as you
have made no net changes, just shuffled things around a bit.

More to come!
=============
This is just part one of a multi-part series.  So far you've gone through
several short iteration loops as you evolve the myface cookbook.  In subsequent
installments, we'll go through more iterations, resulting in the final
end product: <https://github.com/misheska/myface>
