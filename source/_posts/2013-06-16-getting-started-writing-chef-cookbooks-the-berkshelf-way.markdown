---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 1"
date: 2013-06-16 03:49
comments: true
categories: chef
---
* list element with functor item
{:toc}

_Updated December 27th, 2013_

* _Being more prescriptive about the necessary Ruby 1.9.x environment_
* _Bumped VirtualBox from version 4.3.4 to 4.3.6_
* _Bumped vagrant-berkshelf plugin from version 1.3.6 to 1.3.7_
* _Bumped vagrant-omnibus plugin from version 1.1.2 to 1.2.1_
* _Added alternate command lines for Windows, as necessary_
* _Debate on symbols vs strings is an unnecessary distraction, removed this section_
* _Per Dan Patrick introducing the concept of `cookbook_file` before `template`, as this was confusing_

_Updated December 15th, 2013_

* _Bumped CentOS basebox version from 6.4 to 6.5_
* _Added note about issue with Vagrant 1.4.0_
* _Bumped VirtualBox from version 4.2.18 to 4.3.4_
* _Bumped Vagrant from version 1.3.1 to 1.3.5_
* _Bumped vagrant-berkshelf plugin from version 1.3.3 to 1.3.6_
* _Bumped vagrant-omnibus plugin from version 1.1.1 to 1.1.2_

_Updated September 9th, 2013_

* _Bumped VirtualBox from version 4.2.16 to 4.2.18_
* _Bumped berkshelf from version 2.0.9 to 2.0.10_
* _Bumped vagrant from version 1.2.7 to 1.3.1_
* _Bumped vagrant-omnibus plugin from version 1.1.0 to 1.1.1_

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

* [Install VirtualBox 4.x](http://virtualbox.org).  VirtualBox 4.3.6 was tested
for this post.

* [Install Vagrant 1.4.1](http://vagrantup.com) (or higher).  Vagrant 1.4.1 was tested for this post. 

* Set up a [sane Ruby 1.9.x environment](/blog/2013/12/26/set-up-a-sane-ruby-cookbook-authoring-environment-for-chef/) for Chef cookbook authoring

* Install Berkshelf

```
$ gem install berkshelf --no-ri --no-rdoc
Fetching: berkshelf-2.0.10.gem (100%)
Successfully installed berkshelf-2.0.10
1 gem installed

$ berks -v
Berkshelf (2.0.10)

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
Installed the plugin 'vagrant-berkshelf (1.3.7)'!
```

* Install the vagrant-omnibus plugin (1.1.0 or higher)

```
$ vagrant plugin install vagrant-omnibus
Installing the 'vagrant-omnibus' plugin.  This can take a few minutes...
Installed the plugin 'vagrant-omnibus (1.2.1)'!
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

Run `bundle install` in the newly created cookbook directory to install the
necessary Gem dependencies:

    $ cd myface
    $ bundle install
    Fetching gem metadata from https://rubygems.org/.......
    Fetching additional metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.9)
    Using multi_json (1.8.2)
    Using activesupport (3.2.16)
    . . .
    Using berkshelf (2.0.10)
    Using mixlib-shellout (1.3.0)
    Using net-scp (1.1.2)
    Using safe_yaml (0.9.7)
    Using test-kitchen (1.1.1)
    Using kitchen-vagrant (0.14.0)
    Using bundler (1.5.0)
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
    vagrant-omnibus (1.2.1)
    ...

The `vagrant-omnibus` plugin hooks into Vagrant and allows you to specify
the version of the Chef Omnibus package you want installed using the
`omnibus.chef_version` key

Edit the Vagrantfile generated by the `berks cookbook` command to use
a VirtualBox template that does not have a version of Chef provisioned.
Then, specify that you want your image to always use the latest version
of Chef via the `config.omnibus.chef_version` config option and replace the
legacy `config.ssh.max_tries` and `config.ssh.timeout` settings with 
`config.vm.boot_timeout`.

{% codeblock myface/Vagrantfile lang:ruby %}
Vagrant.configure("2") do |config|
  config.vm.hostname = "myface-berkshelf"
  config.vm.box = "centos65"
  config.vm.box_url = "https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box"
  config.omnibus.chef_version = :latest
  config.vm.network :private_network, ip: "33.33.33.10"
  config.vm.boot_timeout = 120
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

NOTE: Vagrant 1.3.0 deprecated the `config.ssh.max_tries` and 
`config.ssh.timeout` settings that are inserted in the Vagrantfile by
Berkshelf.  Until Berkshelf is updated for these changes to vagrant, you'll
need to remove these settings from the `Vagrantfile` and replace them with
`config.vm.boot_timeout` as above.  If you are using vagrant 1.2.x, keep the
`config.ssh.max_tries` and `config.ssh.timeout` settings, as the new
`config.vm.boot_timeout` setting is not valid in the older version of vagrant.

Run `vagrant up` to start up the virtual machine and to test the stub MyFace
cookbook you just created:

    $ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    [default] Box 'centos65' was not found. Fetching box from specified URL for
    the provider 'virtualbox'. Note that if the URL does not have
    a box for this provider, you should interrupt Vagrant now and add
    the box yourself. Otherwise Vagrant will attempt to download the
    full box prior to discovering this error.
    Downloading box from URL: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box
    Extracting box...te: 6881k/s, Estimated time remaining: 0:00:01)
    Successfully added box 'centos65' with provider 'virtualbox'!
    [default] Importing base box 'centos65'...
    [default] Matching MAC address for NAT networking...
    [default] Setting the name of the VM...
    [default] Clearing any previously set forwarded ports...
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20131228-44316-176rdqc-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] Booting VM...
    [default] Waiting for machine to boot. This may take a few minutes...
    [default] Machine booted and ready!
    [default] Setting hostname...
    [default] Configuring and enabling network interfaces...
    [default] Mounting shared folders...
    [default] -- /vagrant
    [default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Installing Chef 11.8.2 Omnibus package...
    [default] Downloading Chef 11.8.2 for el...
    [default] downloading https://www.opscode.com/chef/metadata?v=11.8.2&prerelease=false&p=el&pv=6&m=x86_64
      to file /tmp/install.sh.2993/metadata.txt
    trying wget...
    [default] url	https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
    md5	10f3d0da82efa973fe91cc24a6a74549
    sha256	044558f38d25bbf75dbd5790ccce892a38e5e9f2a091ed55367ab914fbd1cfed
    [default] downloaded metadata file looks valid...
    [default] downloading https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
      to file /tmp/install.sh.2993/chef-11.8.2.x86_64.rpm
    trying wget...
    [default] Checksum compare with sha256sum succeeded.
    [default] Installing Chef 11.8.2
    installing with rpm...
    [default] warning:
    [default] /tmp/install.sh.2993/chef-11.8.2.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
    [default] Preparing...
    [default] ##################################################
    [default]
    [default] chef
    [default] #
    [default]
    [default] Thank you for installing Chef!
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-12-28T13:42:49-08:00] INFO: Forking chef instance to converge...
    [2013-12-28T13:42:49-08:00] INFO: *** Chef 11.8.2 ***
    [2013-12-28T13:42:49-08:00] INFO: Chef-client pid: 3368
    [2013-12-28T13:42:50-08:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-12-28T13:42:50-08:00] INFO: Run List is [recipe[myface::default]]
    [2013-12-28T13:42:50-08:00] INFO: Run List expands to [myface::default]
    [2013-12-28T13:42:50-08:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-12-28T13:42:50-08:00] INFO: Running start handlers
    [2013-12-28T13:42:50-08:00] INFO: Start handlers complete.
    [2013-12-28T13:42:50-08:00] INFO: Chef Run complete in 0.023677599 seconds
    [2013-12-28T13:42:50-08:00] INFO: Running report handlers
    [2013-12-28T13:42:50-08:00] INFO: Report handlers complete
    [2013-12-28T13:42:49-08:00] INFO: Forking chef instance to converge...

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
    [default] Running cleanup tasks for 'chef_solo' provisioner...

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

group 'myface'

user 'myface' do
  group 'myface'
  system true
  shell '/bin/bash'
end
{% endcodeblock %}

Save `recipes/default.rb` and re-run `vagrant provision` to create the
myface user on your test virtual machine:

    $ vagrant provision
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20131228-44581-4bhc9d-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Chef 11.8.2 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-12-28T13:57:59-08:00] INFO: Forking chef instance to converge...
    [2013-12-28T13:57:59-08:00] INFO: *** Chef 11.8.2 ***
    [2013-12-28T13:57:59-08:00] INFO: Chef-client pid: 3845
    [2013-12-28T13:57:59-08:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-12-28T13:57:59-08:00] INFO: Run List is [recipe[myface::default]]
    [2013-12-28T13:57:59-08:00] INFO: Run List expands to [myface::default]
    [2013-12-28T13:57:59-08:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-12-28T13:57:59-08:00] INFO: Running start handlers
    [2013-12-28T13:57:59-08:00] INFO: Start handlers complete.
    [2013-12-28T13:57:59-08:00] INFO: group[myface] created
    [2013-12-28T13:57:59-08:00] INFO: user[myface] created
    [2013-12-28T13:57:59-08:00] INFO: Chef Run complete in 0.157055758 seconds
    [2013-12-28T13:57:59-08:00] INFO: Running report handlers
    [2013-12-28T13:57:59-08:00] INFO: Report handlers complete
    [2013-12-28T13:57:59-08:00] INFO: Forking chef instance to converge...

You should expect to see the Chef run complete with no errors.  Notice
that it also creates `group[myface]` and `user[myface]`.

Testing Iteration #1
--------------------

Verify that Chef actually created the myface user on our test virtual
machine by running the following on Mac OS X/Linux:

    $ vagrant ssh -c "getent passwd myface"
    myface:x:497:501::/home/myface:/bin/bash

On Windows, add two extra parameters to the ssh invocation to squelch some
error messages:

    > vagrant ssh -c "getent passwd myface" -- -n -T
    myface:x:497:501::/home/myface:/bin/bash

The extra `-n` and `-T` parameters for Windows are additional ssh parameters
to prevent trying to read from stdin and to suppress an error message
that a pseudo terminal can't be allocated, respectively.  On Windows,
vagrant runs as a service, and windows services do not have console
sessions attached which ssh assumes, so you'll see some errors on Windows
if you don't use these extra parameters.

We use `vagrant ssh -c` to run a command on our test virtual machine.  The
`getent` command can be used to query all user databases.  In this
case we're looking for `myface`, and it exists!

Because we are using well-defined resources that are completely
[idempotent](http://en.wikipedia.org/wiki/Idempotence), you should notice
that if you run `vagrant provision` again, the Chef run executes more quickly
and it does not try to re-create the user/group it already created.

    $ vagrant provision
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20131228-44581-4bhc9d-default'
    [Berkshelf] Using myface (0.1.0)
    [default] Chef 11.8.2 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-12-28T14:01:18-08:00] INFO: Forking chef instance to converge...
    [2013-12-28T14:01:18-08:00] INFO: *** Chef 11.8.2 ***
    [2013-12-28T14:01:18-08:00] INFO: Chef-client pid: 4378
    [2013-12-28T14:01:18-08:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-12-28T14:01:18-08:00] INFO: Run List is [recipe[myface::default]]
    [2013-12-28T14:01:18-08:00] INFO: Run List expands to [myface::default]
    [2013-12-28T14:01:18-08:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-12-28T14:01:18-08:00] INFO: Running start handlers
    [2013-12-28T14:01:18-08:00] INFO: Start handlers complete.
    [2013-12-28T14:01:18-08:00] INFO: Chef Run complete in 0.023349135 seconds
    [2013-12-28T14:01:18-08:00] INFO: Running report handlers
    [2013-12-28T14:01:18-08:00] INFO: Report handlers complete
    [2013-12-28T14:01:18-08:00] INFO: Forking chef instance to converge...

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
default['myface']['user'] = 'myface'
default['myface']['group'] = 'myface'
{% endcodeblock %}

In Chef, attributes are a hash of a hash used to override the default settings
on a node.  The first hash is the cookbook name - in our
case we've named our cookbook `'myface'`. The second hash is the name of
our attribute - in this case, we're defining two new attributes: `'user'` and
`'group'`.

`default` implies the use of the [node object](http://docs.opscode.com/chef/essentials_node_object.html)
`node.default` and is a Chef attribute file shorthand.  The following are
equivalent definitions to the ones above:

    node.default['myface']['user'] = 'myface'
    node.default['myface']['group'] = 'myface'

Note that the hash names are defined as strings enclosed by quotes.
In this case, it doesn't matter if you use single or double quotes.  In
Chef source files, double quotes indicate that
[string interpolation](http://rubymonk.com/learning/books/1/chapters/5-strings/lessons/31-string-basics?section=143)
should be performed, replacing special tokens in a string with their
values.  If single quotes are used, no string interpolation is performed.
We'll see more examples of when string interpolation is necessary later
in this artile.

Now that you've created your attribute definitions, edit
`myface/recipes/default.rb` and replace all references to the 'myface' user name
with `node['myface']['user']` and all references to the 'myface' group with
`node['myface']['group']`.  `myface/recipes/default.rb` should now look like
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

group node['myface']['group']

user node['myface']['user'] do
  group node['myface']['group']
  system true
  shell '/bin/bash'
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
result as when you validated Iteration #1 on Mac OS X/Linux:

    $ vagrant ssh -c "getent passwd myface"
    myface:x:497:501::/home/myface:/bin/bash

And on Windows:

    > vagrant ssh -c "getent passwd myface" -- -n -T
    myface:x:497:501::/home/myface:/bin/bash

Iteration #3 - Install the Apache2 Web Server
============================================
Our hot new social networking application, myface, is a web app, so we need
to install a web server.  Let's install the Apache2 web server.

Modify `myface/recipes/default.rb` to include the apache2 cookbook's default
recipe:

    include_recipe 'apache2'

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

group node['myface']['group']

user node['myface']['user'] do
  group node['myface']['group']
  system true
  shell '/bin/bash'
end

include_recipe 'apache2'
{% endcodeblock %}

Since you are loading Apache2 from another cookbook, you need to configure the
dependency in your metadata.  Edit `myface/metadata.rb` and add the `apache2`
dependency at the bottom:

    depends 'apache2', '~> 1.8.0'

This tells Chef that the myface cookbook depends on the apache2 cookbook.
We've also specified a version constraint using the optimistic operator
`~>` to tell our Chef that we want the latest version of the apache2 cookbook
that is greater than 1.8.0 but *not* 1.9.0 or higher.

It is recommended that Chef cookbooks follow a
[Semantic Versioning](http://semver.org/) scheme.  So if you write your
cookbook to use the latest apache2 1.8.x cookbook, if the apache2 cookbook is
ever bumped to a 1.9.x version (or 2.x), it has some public API functionality
that has at least been deprecated.  So you'll want to review the changes and
test before automatically using an apache2 cookbook version 1.9.x or higher.
However, automatic use of any new 1.8.x is perfectly fine, because no
only backwards-compatible bug fixes has been introduced.  Semantic Versioning
guarantees there are no changes in the public APIs.

Your `myface/metadata.rb` file should look like this:

{% codeblock myface/metadata.rb lang:ruby %}
name             'myface'
maintainer       'YOUR_NAME'
maintainer_email 'YOUR_EMAIL'
license          'All rights reserved'
description      'Installs/Configures myface'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.1.0'

depends 'apache2', '~> 1.8.0'
{% endcodeblock %}

Now when you re-run `vagrant provision` it will install apache2 on your
test virtual machine:

    $ vagrant provision
    [Berkshelf] This version of the Berkshelf plugin has not been fully tested on this version of Vagrant.
    [Berkshelf] You should check for a newer version of vagrant-berkshelf.
    [Berkshelf] If you encounter any errors with this version, please report them at https://github.com/RiotGames/vagrant-berkshelf/issues
    [Berkshelf] You can also join the discussion in #berkshelf on Freenode.
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20131228-44581-4bhc9d-default'
    [Berkshelf] Using myface (0.1.0)
    [Berkshelf] Installing apache2 (1.8.14) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [default] Chef 11.8.2 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    ...
    [2013-12-28T14:10:49-08:00] INFO: service[apache2] started
    [2013-12-28T14:10:49-08:00] INFO: template[/etc/httpd/mods-available/deflate.conf] sending restart action to service[apache2] (delayed)
    [2013-12-28T14:10:51-08:00] INFO: service[apache2] restarted
    [2013-12-28T14:10:51-08:00] INFO: Chef Run complete in 38.225601754 seconds
    [2013-12-28T14:10:51-08:00] INFO: Running report handlers
    [2013-12-28T14:10:51-08:00] INFO: Report handlers complete
    [2013-12-28T14:10:12-08:00] INFO: Forking chef instance to converge...

Testing Iteration #3
--------------------

You can verify that the apache2 `httpd` service is running on your berkshelf
virtual machine with the following command on Mac OS X/Linux:

    $ vagrant ssh -c "sudo /sbin/service httpd status"
    httpd (pid  5758) is running.

And on Windows:

    > vagrant ssh -c "sudo /sbin/service httpd status" -- -n -T
    httpd (pid  5758) is running.

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
in `~/.berkshelf/default/cookbooks` (or
`%USERPROFILE%\.berkshelf\default\cookbooks` on Windows)

    Users/misheska/.berkshelf/default/cookbooks
    └── apache2-1.8.14
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

`~/.berkshelf` (or `%USERPROFILE%\.berkshelf` on Windows) is just the default
location where Berkshelf stores data on your local disk.  This location can be
altered by setting the environment variable `BERKSHELF_PATH`.

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

group node['myface']['group']

user node['myface']['user'] do
  group node['myface']['group']
  system true
  shell '/bin/bash'
end

include_recipe 'apache2'

# disable default site
apache_site '000-default' do
  enable false
end

# create apache config
template "#{node['apache']['dir']}/sites-available/myface.conf" do
  source 'apache2.conf.erb'
  notifies :restart, 'service[apache2]'
end

# create document root
directory '/srv/apache/myface' do
  action :create
  recursive true
end

# write site
cookbook_file '/srv/apache/myface/index.html' do
  mode '0644' # forget me to create debugging exercise
end

# enable myface
apache_site 'myface.conf' do
  enable true
end
{% endcodeblock %}

If you're familiar with Chef and configuring a web app via apache2, nothing
here should be too surprising.  But if not, spend some time reading up on
the resource references at <http://docs.opscode.com>

Note our first use of [string interpolation](http://rubymonk.com/learning/books/1/chapters/5-strings/lessons/31-string-basics?section=143)
on line 26:

    template "#{node['apache']['dir']}/sites-available/myface.conf" do

We need to enclose the template string in double-quotes because we're
using the Ruby `#{}` operator to indicate that we want to perform string
interpolation and evaluate the value of `node['apache']['dir']` inserting the
evaluated value in the string.

Create a file to contain our web site content as
`myface/files/default/index.html`.  
Chef looks for files in the `files` subtree by default.  A subdirectory
level underneath `files` is used to specify platform-specific files.
Files in `files/default` are used with every platform.   During the Chef
run this file is copied to the target node in the location
specified in the [cookbook_file](http://docs.opscode.com/resource_cookbook_file.html)
resource (see line 38 of `myface/recipes/default.rb`).

{% codeblock myface/files/default/index.html lang:ruby %}
Welcome to MyFace!
{% endcodeblock %}

With Chef, you can also create config files from templates using
[ERB](http://ruby-doc.org/stdlib-1.9.3/libdoc/erb/rdoc/ERB.html), a
Ruby templating system.  When the template `.erb` file is processed by Chef,
it will replace any token bracketed by a `<%=` `%>` tag with the value
of the Ruby expression inside the token (like `node[:hostname]`).  (Unlike
with a `cookbook_file` whose content is static and is never manipulated
by Chef).

Chef expects ERB template files to be in the `templates` subirectory of a
cookbook.  A subdirectory level underneath `templates` is used to specify
platorm-specific files.  Files in the `templates/default` subdirectory are
used with every platform.

Create a new template file called
`myface/templates/default/apache2.conf.erb` which will become the
file `.../sites-available/myface.conf` on our test virtual machine
(refer to `myface/recipes/default.rb` above):

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node['hostname'] %>

Alias / /srv/apache/myface/

<Directory /srv/apache/myface>
	Options FollowSymLinks +Indexes
	Allow from All
</Directory>
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

Also, note the difference between a `cookbook_file` and a `template`.  Run this
command on Mac OS X/Linux to print out `index.html` that was copied to the node during the Chef run.

    $ vagrant ssh -c "cat /srv/apache/myface/index.html"
    Welcome to MyFace!
    Connection to 127.0.0.1 closed.

And on Windows:

    > vagrant ssh -c "cat /srv/apache/myface/index.html" -- -n -T
    Welcome to MyFace!
    Connection to 127.0.0.1 closed.

Note that `index.html` is copied verbatim with no changes to the file.

By comparison, in a `template` any ERB tokens are replaced when they are
copied to the node.  Look at the resultant `myface.conf` on the node
with the following command on Mac OS X/Linux:

    $ vagrant ssh -c "cat /etc/httpd/sites-available/myface.conf"
    # Managed by Chef for myface-berkshelf

    Alias / /srv/apache/myface/

    <Directory /srv/apache/myface>
     	Options FollowSymLinks +Indexes
     	Allow from All
    </Directory>
    Connection to 127.0.0.1 closed.

and with the following command on Windows:

    > vagrant ssh -c "cat /etc/httpd/sites-available/myface.conf" -- -n -T

The ERB token `<%= node['hostname'] %>` was replaced by the evaluated string
`myface-berkshelf` during the Chef run.

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

group node['myface']['group']

user node['myface']['user'] do
  group node['myface']['group']
  system true
  shell '/bin/bash'
end

include_recipe 'apache2'

# disable default site
apache_site '000-default' do
  enable false
end

# create apache config
template "#{node['apache']['dir']}/sites-available/myface.conf" do
  source 'apache2.conf.erb'
  notifies :restart, 'service[apache2]'
end

# create document root
directory '/srv/apache/myface' do
  action :create
  recursive true
end

# write site
cookbook_file '/srv/apache/myface/index.html' do
  mode '0644'
end

# enable myface
apache_site 'myface.conf' do
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

include_recipe 'myface::webserver'
{% endcodeblock %}

Converge your node again to make sure there are no syntax errors:

    $ vagrant provision

Let's eliminate some more of the duplication that crept in while we were
working on things.  Edit `myface/attributes/default.rb`

{% codeblock myface/attributes/default.rb lang:ruby %}
default['myface']['user'] = 'myface'
default['myface']['group'] = 'myface'
default['myface']['name'] = 'myface'
default['myface']['config'] = 'myface.conf'
default['myface']['document_root'] = '/srv/apache/myface'
{% endcodeblock %}

NOTE: With Chef 11, it is now possible to nest attributes, like so:

    node.default['app']['name'] = 'my_app'
    node.default['app']['document_root'] = "/srv/apache/#{node['app']['name']}"

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

group node['myface']['group']

user node['myface']['user'] do
  group node['myface']['group']
  system true
  shell '/bin/bash'
end

include_recipe 'apache2'

# disable default site
apache_site '000-default' do
  enable false
end

# create apache config
template "#{node['apache']['dir']}/sites-available/#{node['myface']['config']}" do
  source 'apache2.conf.erb'
  notifies :restart, 'service[apache2]'
end

# create document root
directory "#{node['myface']['document_root']}" do
  action :create
  recursive true
end

# write site
cookbook_file "#{node['myface']['document_root']}/index.html" do
  mode '0644'
end

# enable myface
apache_site "#{node['myface']['config']}" do
  enable true
end
{% endcodeblock %}

Converge your node to make sure there are no syntax errors:

    $ vagrant provision

Add tokens to the `templates/default/apache2.conf.erb` ERB template
file so they will be replaced with the value of the
`node['myface']['document_root']` attribute during the Chef run.

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node['hostname'] %>

Alias / <%= node['myface']['document_root'] %>/

<Directory <%= node['myface']['document_root'] %>>
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
name             'myface'
maintainer       'Mischa Taylor'
maintainer_email 'mischa@misheska.com'
license          'Apache 2.0'
description      'Installs/Configures myface'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '1.0.0'

depends 'apache2', '~> 1.8.0'
{% endcodeblock %}

Configure Berkshelf
-------------------

In order to deploy to your production server (instead of just locally with
vagrant), you'll need to add a section to your Berkshelf config file with
your production Chef settings.  Run the following command to create a default
Berkshelf configuration file:

    $ berks configure

It will prompt you for a bunch of server settings.  Since I'm using hosted Chef
to manage my servers, my settings are as follows (yours will be 
slightly different):

    $ berks configure
    Enter value for chef.chef_server_url (default: 'http://localhost:4000'):  https://api.opscode.com/organizations/misheska
    Enter value for chef.node_name (default: 'Ruby.local'):  misheska
    Enter value for chef.client_key (default: '/etc/chef/client.pem'):  /Users/misheska/.chef/misheska.pem
    Enter value for chef.validation_client_name (default: 'chef-validator'):  misheska-validator
    Enter value for chef.validation_key_path (default: '/etc/chef/validation.pem'):  /Users/misheska/.chef/misheska-validator.pem
    Enter value for vagrant.vm.box (default: 'Berkshelf-CentOS-6.3-x86_64-minimal'):  centos65
    Enter value for vagrant.vm.box_url (default: 'https://dl.dropbox.com/u/31081437/Berkshelf-CentOS-6.3-x86_64-minimal.box'):  https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box
    Config written to: '/Users/misheska/.berkshelf/config.json'

The config file is located in `$HOME/.berkshelf/config.json` (or inx
`%USERPROFILE%/.berkshelf/config.json` on Windows).

Here's what my `$HOME/.berkshelf/config.json` file looks like:

    {
      "chef":{
        "chef_server_url":"https://api.opscode.com/organizations/misheska",
        "validation_client_name":"misheska-validator",
        "validation_key_path":"/Users/misheska/.chef/misheska-validator.pem",
        "client_key":"/Users/misheska/.chef/misheska.pem",
        "node_name":"misheska"
      },
      "cookbook":{
        "copyright":"Mischa Taylor",
        "email":"mischa@misheska.com",
        "license":"Apache 2.0"
      },
      "allowed_licenses":[],
      "raise_license_exception":false,
      "vagrant":{
        "vm":{
          "box":"centos65",
          "box_url":"https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box",
          "forward_port":{},
          "network":{"bridged":false,"hostonly":"33.33.33.10"},
          "provision":"chef_solo"}
      },
      "ssl":{
        "verify":false
      }
    }

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
          "box": "centos65",
          "box_url":"https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box",
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
    Using apache2 (1.8.14)
    Uploading myface (1.0.0) to: 'https://api.opscode.com:443/organizations/misheska'
    Uploading apache2 (1.8.14) to: 'https://api.opscode.com:443/organizations/misheska'

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
    Starting Chef Client, version 11.8.2
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
