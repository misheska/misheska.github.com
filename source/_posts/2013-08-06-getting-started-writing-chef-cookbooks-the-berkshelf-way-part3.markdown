---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 3"
date: 2013-08-06 12:46
comments: true
categories: chef
---
* list element with functor item
{:toc}

_Updated December 29, 2013_

* _Bumped Test Kitchen from 1.0.0.beta.3 to 1.1.1_
* _Bumped CentOS to version 6.5_
* _Per Kelly Setzer, updated os check with recent RSpec updates_

_Updated September 10, 2013_

* _Bumped VirtualBox images from version 4.2.16 to 4.2.18_
* _Bumped Vagrant from version 1.2.7 to 1.3.1_

_Updated September 1, 2013_

* _Bumped Test Kitchen from 1.0.0.beta.2 to 1.0.0.beta.3_

This is the third article in a series on writing Opscode Chef cookbooks the
Berkshelf Way.  Here's a link to [Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/) and
[Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/).  The source code examples covered in this
article can be found on Github: <https://github.com/misheska/myface>

In this installment, we're going to learn how to use Test Kitchen to 
automate all the verification steps we did by hand for each iteration in
[Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/) and
[Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/).  If not anything else, it's worth learning
Test Kitchen because OpsCode, the company that makes Chef, has encouraged the
use of Test Kitchen to verify community cookbooks.

Test Kitchen is built on top of vagrant and supplements the `Vagrantfile` file
you have been using so far in this series to do local automated testing.  The
main benefit to Test Kitchen is that it makes it easy to run tests on multiple
platforms in parallel, which is more difficult to do with just a `Vagrantfile`.
We'll be showcasing this aspect of Test Kitchen by ensuring that Myface works
on both the CentOS 6.4 and Ubuntu 12.04 Linux distributions.

Iteration #13 - Install Test Kitchen
====================================
Edit `myface/Gemfile` and add the following lines to load the
Test Kitchen gems:

    gem 'test-kitchen'
    gem 'kitchen-vagrant'

Depending on when you went through this article series, your `Gemfile`
may already have these additions.  After editing, your `myface/Gemfile`
should look like the following after editing:

{% codeblock myface/Gemfile lang:ruby %}
source 'https://rubygems.org'

gem 'berkshelf'
gem 'test-kitchen'
gem 'kitchen-vagrant'
{% endcodeblock %}

After you have updated the `Gemfile` run `bundle install` to download the
test-kitchen gem and all its dependencies:

    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.9)
    Using multi_json (1.8.2)
    Using activesupport (3.2.16)
    ...
    Installing test-kitchen (1.1.1)
    Using bundler (1.5.1)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

Testing Iteration #13 - Show the Test Kitchen version
-----------------------------------------------------

If everything worked properly you should be able to run the `kitchen --version`
command to see your installed Test Kitchen's version information

    $ kitchen --version
    Test Kitchen version 1.1.1

Iteration #14 - Create a Kitchen YAML file
==========================================

In order to use Test Kitchen on a cookbook, first you need to add a few more
dependencies and create a template Kitchen YAML file.  Test Kitchen makes this
easy by providing the `kitchen init` command to perform all these
initialization steps automatically 

    $ kitchen init
          create  .kitchen.yml
          append  Thorfile
          create  test/integration/default
          append  .gitignore
          append  .gitignore
          append  Gemfile
          append  Gemfile
    You must run 'bundle install' to fetch any new gems.

Since `kitchen init` modified your Gemfile, you need to re-run `bundle install`
(as suggested above) to pick up the new gem dependencies:

    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching additional metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.9)
    Using multi_json (1.8.2)
    Using activesupport (3.2.16)
    ...
    Using safe_yaml (0.9.7)
    Using test-kitchen (1.1.1)
    Installing kitchen-vagrant (0.14.0)
    Using bundler (1.5.1)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

Most importantly, this new `bundle install` pass installed the
`kitchen-vagrant` vagrant driver for Test Kitchen.

Now that you have created a `.kitchen.yml` Kitchen YAML file and loaded all
the necessary gems, let's customize the file so that we'll use 
to verify that the Myface cookbook works on CentOS 6.4, like we were doing in
the previous installments with the `Vagrantfile`.

{% codeblock myface/.kitchen.yml %}

---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
- name: centos65
  driver_config:
    box: centos65
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes:
    mysql:
      server_root_password: "rootpass"
      server_debian_password: "debpass"
      server_repl_password: "replpass"

{% endcodeblock %}

Everything in the YAML file should be straightforward to understand, except
perhaps the `attributes` item in the `suites` stanza.  These values 
came from the `Vagrantfile` we used in the previous installments of this
series.  Here's an excerpt from the `Vagrantfile` - at the end are some
values that Berkshelf initialzed (which we used in
[Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/)).

    ...
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

Those `Vagrantfile` attributes were just converted into a format that the
Test Kitchen YAML file format finds acceptable.

You can add even more Vagrantfile customizations to your `kitchen.yml` file
if you like.  For example, you can assign a host-only network IP so you can
look at the MyFace website with a browser on your host.  Add the following
`network:` block to a platform's `driver_config:`:

    ...
    driver_config:
      network:
      - ["private_network", {ip: "33.33.33.10"}]
    ...

After adding this block your `.kitchen.yml` should look like this:

{% codeblock myface/.kitchen.yml %}

---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
- name: centos65
  driver_config:
    box: centos65
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box
    network:
    - ["private_network", {ip: "33.33.33.10"}]

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes:
    mysql:
      server_root_password: "rootpass"
      server_debian_password: "debpass"
      server_repl_password: "replpass"
{% endcodeblock %}

For more information on the `kitchen-vagrant` settings, refer to the
README.md file for `kitchen-vagrant` at <https://github.com/opscode/kitchen-vagrant/blob/master/README.md>

Testing Iteration #14 - Provision with Test Kitchen
---------------------------------------------------

You can do nearly everything that you were doing with vagrant just using Test
Kitchen.  The Test Kitchen equivalent of the `vagrant up` command is 
`kitchen converge`.  Try running the `kitchen converge` command now to verify
that your `.kitchen.yml` file is valid.  When you run `kitchen converge` it will
spin up a CentOS 6.5 vagrant test node instance and use Chef Solo to provision
the MyFace cookbook on the test node:

    $ kitchen converge 
    -----> Starting Kitchen (v1.1.1)
    -----> Creating <default-centos65>...
           Bringing machine 'default' up with 'virtualbox' provider...
           [default] Importing base box 'centos65'...
           [default] Matching MAC address for NAT networking...
           [default] Setting the name of the VM...
           [default] Clearing any previously set forwarded ports...
           [Berkshelf] Skipping Berkshelf with --no-provision
           [default] Fixed port collision for 22 => 2222. Now on port 2200.
           [default] Clearing any previously set network interfaces...
           [default] Preparing network interfaces based on configuration...
           [default] Forwarding ports...
           [default] -- 22 => 2200 (adapter 1)
           [default] Running 'pre-boot' VM customizations...
           [default] Booting VM...
           [default] Waiting for machine to boot. This may take a few minutes...
           [default] Machine booted and ready!
           [default] Setting hostname...
           Vagrant instance <default-centos65> created.
           Finished creating <default-centos65> (0m51.76s).
    -----> Converging <default-centos65>...
           Preparing files for transfer
           Resolving cookbook dependencies with Berkshelf...
           Removing non-cookbook files before transfer
    -----> Installing Chef Omnibus (true)
           downloading https://www.opscode.com/chef/install.sh
             to file /tmp/install.sh
           trying wget...
           Downloading Chef  for el...
           downloading https://www.opscode.com/chef/metadata?v=&prerelease=false&p=el&pv=6&m=x86_64
             to file /tmp/install.sh.2158/metadata.txt
           trying wget...
           url	https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
           md5	10f3d0da82efa973fe91cc24a6a74549
           sha256	044558f38d25bbf75dbd5790ccce892a38e5e9f2a091ed55367ab914fbd1cfed
           downloaded metadata file looks valid...
           downloading https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-11.8.2-1.el6.x86_64.rpm
             to file /tmp/install.sh.2158/chef-.x86_64.rpm
           trying wget...
           Checksum compare with sha256sum succeeded.
           Installing Chef
           installing with rpm...
           warning: /tmp/install.sh.2158/chef-.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
    Preparing...                #####  ########################################### [100%]
       1:chef                          ########################################### [100%]
           Thank you for installing Chef!
           Transfering files to <default-centos65>
           [2013-12-29T11:44:52-08:00] INFO: Forking chef instance to converge...
           Starting Chef Client, version 11.8.2
           [2013-12-29T11:44:52-08:00] INFO: *** Chef 11.8.2 ***
           [2013-12-29T11:44:52-08:00] INFO: Chef-client pid: 2257
           [2013-12-29T11:44:53-08:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
           [2013-12-29T11:44:53-08:00] INFO: Run List is [recipe[myface::default]]
           ...
           Recipe: apache2::default
             * service[apache2] action restart[2013-12-29T11:47:41-08:00] INFO: Processing service[apache2] action restart (apache2::default line 210)
           [2013-12-29T11:47:43-08:00] INFO: service[apache2] restarted

               - restart service service[apache2]

           [2013-12-29T11:47:43-08:00] INFO: Chef Run complete in 170.575449983 seconds
           [2013-12-29T11:47:43-08:00] INFO: Running report handlers
           [2013-12-29T11:47:43-08:00] INFO: Report handlers complete
           Chef Client finished, 100 resources updated
           Finished converging <default-centos65> (3m43.06s).
    -----> Kitchen is finished. (4m35.40s)


To display the results of the Chef Run, type in the `kitchen list` command:

    $ kitchen list
    Instance           Driver   Provisioner  Last Action
    default-centos65   Vagrant  Chef Solo    Converged

If the run succeeded, it should display `Converged` in the `Last Action` field.

The Test Kitchen equivalent of the `vagrant ssh` command is `kitchen login`.
Since Test Kitchen supports multiple instances, you will need to pass in
the instance name for which you wish to login as a parameter (which you can
get from the `kitchen list` output).  We want to login to the CentOS 6.5
instance (the only instance for now), so type in the command
`kitchen login default-centos65` to login:

    $ kitchen login default-centos65
    Last login: Sun Dec 29 13:16:33 2013 from 10.0.2.2
    Welcome to your Packer-built virtual machine.
    [vagrant@default-centos65 ~]$

Now you can poke around in the image the same way you did with `vagrant ssh`,
for example, verifying that the httpd server is running:

    [vagrant@default-centos65 ~]$ sudo /sbin/service httpd status
    httpd (pid  4410) is running...

After you are done working in the test instance, make sure to run the
`exit` command to log out so that you return back to your host prompt:

    [vagrant@default-centos65 ~]$ exit
    logout
    Connection to 127.0.0.1 closed.

Should you need it, the Test Kitchen equivalent of `vagrant destroy` is
`kitchen destroy`.  If you make a change to the chef cookbook and
want to re-deploy, the Test Kitchen equivalent of `vagrant provision` is
`kitchen converge`.

Since you added a private IP for you instance, you can also view the
MyFace website on your host with your favorite web browser:

<http://33.33.33.10>

![Welcome to MyFace](/images/welcometomyface2.png)

Iteration #15 - Provisioning Ubuntu
===================================

We haven't really made use of any unique Test Kitchen features yet, let's
start now.  We'll also deploy our cookbook locally to Ubuntu 12.04 for
testing, in addition to CentOS 6.5.

Edit `.kitchen.yml` and add a reference to a Ubuntu 12.04 basebox alongside
the existing CentOS 6.5 basebox in the `platforms` stanza:

    - name: ubuntu1204
      driver_config:
        box: ubuntu1204
        box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/ubuntu1204.box
        network:
        - ["private_network", {ip: "33.33.33.11"}]

After editing, your `.kitchen.yml` file should resemble the following:

{% codeblock myface/.kitchen.yml %}

---
driver:
  name: vagrant

provisioner:
  name: chef_solo

platforms:
- name: centos65
  driver_config:
    box: centos65
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/centos65.box 
    network:
    - ["private_network", {ip: "33.33.33.10"}]
- name: ubuntu1204
  driver_config:
    box: ubuntu1204
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox4.3.6/ubuntu1204.box
    network:
    - ["private_network", {ip: "33.33.33.11"}]

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes:
    mysql:
      server_root_password: "rootpass"
      server_debian_password: "debpass"
      server_repl_password: "replpass"
{% endcodeblock %}

Before we run `kitchen converge` to do a Chef run, we need to fix our cookbook
so it will run successfully on Ubuntu 12.04.  If you tried to deploy now you
would notice that the MyFace cookbook would fail to deploy to Ubuntu 12.04
successfully due to a reference to the `php-mysql` package in 
`myface/receipes/webserver.rb`.

    ... 
    include_recipe 'apache2'
    include_recipe 'apache2::mod_php5'

    package 'php-mysql' do
      action :install
      notifies :restart, 'service[apache2]'
    end
    ...

On Ubuntu the package name should be `php5-mysql` instead of `php-mysql`.

As with most issues in the Chef world, there's a cookbook for that!
The Opscode `php` cookbook has conditionals to reference the correct name
for the `php-mysql` package on a number of platforms.

Edit `myface/metadata.rb` and add a reference to the latest version of the
`php` cookbook (currently 1.3.10):

{% codeblock myface/metadata.rb %}
name             'myface'
maintainer       'Mischa Taylor'
maintainer_email 'mischa@misheska.com'
license          'Apache 2.0'
description      'Installs/Configures myface'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '2.0.0'

depends 'apache2', '~> 1.8.0'
depends 'mysql', '~> 4.0.0'
depends 'database', '~> 1.6.0'
depends 'php', '~> 1.3.0'
{% endcodeblock %}

In `myface/recipes/webserver.rb` replace the `package "php-mysql" do ... end`
block with the following reference:

    include_recipe 'php::module_mysql'

After editing, `myface/recipes/webserver.rb` should look like this:

{% codeblock myface/recipes/webserver.rb %}
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
include_recipe 'apache2::mod_php5'

include_recipe 'php::module_mysql'

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
  mode '0755'
  recursive true
end

# write site
template "#{node['myface']['document_root']}/index.php" do
  source 'index.php.erb'
  mode '0644'
end

# enable myface
apache_site "#{node['myface']['config']}" do
  enable true
end
{% endcodeblock %}

Testing Iteration #15 - Deploy locally to Ubuntu 12.04
------------------------------------------------------

Now that we've fixed up our cookbook to work on Ubuntu 12.04, let's test it
out!  Run `kitchen list` to display the list of Test Kitchen instances:

    Instance             Driver   Provisioner  Last Action
    default-centos65     Vagrant  Chef Solo    Converged
    default-ubuntu1204   Vagrant  Chef Solo    <Not Created>

Notice that after editing the `.kitchen.yml` we now have an Ubuntu 12.04
instance called `default-ubuntu1204` and it is in the `<Not Created>`
state.

Go ahead setup the Ubuntu 12.04 instance by running `kitchen converge` again:

    $ kitchen converge default-ubuntu1204

Note that this time we added an optional instance parameter so that Test
Kitchen only performs the action against the specified instance.  If you do
not specify this parameter, it defaults to `all`, running the command against
all instances.  After about 5-10 minutes or so, you should observe that Test
Kitchen downloaded an Ubuntu 12.04 basebox, booted a VM with the basebox, and
successfully deployed our chef cookbook.

Run the `kitchen list` command again to verify that the Ubuntu 12.04 instance
is now in the `Set Up` state as well, showing that there were no errors:

    Instance             Driver   Provisioner  Last Action
    default-centos65     Vagrant  Chef Solo    Converged
    default-ubuntu1204   Vagrant  Chef Solo    Converged

You just fixed an error with the MyFace cookbook that prevented deployment to
Ubuntu 12.04, and verified that the cookbook correctly deploys to both 
Ubuntu 12.04 and Centos 6.5.

Use the `kitchen login` command to ssh into each instance and poke around
if you like.  You now have two local vagrant VMs instantiated to play with!

    $ kitchen login default-ubuntu1204
    Welcome to Ubuntu 12.04.3 LTS (GNU/Linux 3.8.0-29-generic x86_64)

     * Documentation:  https://help.ubuntu.com/
    Last login: Sun Dec 29 13:30:36 2013 from 10.0.2.2
    $ [...poke around, run some commands...]
    $ exit
    Connection to 127.0.0.1 closed.

    $ kitchen login default-centos65
    Last login: Sun Dec 29 13:21:10 2013 from 10.0.2.2
    Welcome to your Packer-built virtual machine.
    [vagrant@default-centos65 ~]$ [...poke around, run some commands...]
    [vagrant@default-centos65 ~]$ exit
    logout
    Connection to 127.0.0.1 closed.

You can view the websites for each instance by viewing the appropriate
private IP

    CentOS 6.5:   http://33.33.33.10
    Ubuntu 12.04: http://33.33.33.11


Iteration #16 - Writing your first Serverspec test
==================================================

While it's really helpful to know now that the Myface cookbook will converge on
both a CentOS 6.5 and Ubuntu 12.04 setup, we haven't written any tests yet.
Let's do that.

It's helpful to know that Test Kitchen was designed as a framework for
post-convergence system testing.  You are supposed to set up a bunch of test
instances, perform a Chef run to apply your cookbook's changes to them, then
when this is process is complete your tests can inspect the state of each
test instance after the Chef run is finished.  This is how we tested our
nodes by hand in [Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)
and [Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/).  Now we are going to automate the process.

NOTE: It is a [testing anti-pattern to rely too much on system tests, even if they are automated](http://watirmelon.com/2012/01/31/introducing-the-software-testing-ice-cream-cone/)
so make sure you are judicious in your use of system tests.  It can be
difficult to maintain a lot of system tests over time and keep them relevant.
The tests you performed by hand in [Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)
and [Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/)
to verify MyFace are at just the right level of detail for a system test.
Do just enough to verify that the system works correctly after it was
configured.  In a future post, I'll cover unit tests in more detail using
[Chefspec](http://acrmp.github.io/chefspec/) and
[Guard](https://github.com/guard/guard).  For now, let's focus on system tests.

Test Kitchen finds tests by following a directory naming convention.  When
you installed Test Kitchen, it created the `test/integration/default`
subdirectory underneath your main cookbook.  It looks for test code in the
following directory underneath `test/integration`:

    <COOKBOOOK-PATH>/test/integration/<TEST-SUITE-NAME>/<PLUGIN-NAME>

A collection of tests is called a [test suite](http://en.wikipedia.org/wiki/Test_suite).  Following Test Kitchen install's lead, we'll just call our first
suite of tests `default`.  Test Kitchen has a number of plugins which will
install and setup any components necessary for running tests.  We'll be using
the `severspec` plugin for our tests.  So you will be placing your test files
in the following directory:

    myface/test/integration/default/serverspec

Create the `myface/test/integration/default/serverspec` subdirectory now.

To start, we need to add a Ruby helper script which loads our Test Kitchen
plugin.  We'll call it
`myface/test/integration/default/serverspec/spec_helper.rb`:

{% codeblock myface/test/integration/default/serverspec/spec_helper.rb lang:ruby %}
require 'serverspec'
require 'pathname'

include Serverspec::Helper::Exec
include Serverspec::Helper::DetectOS

RSpec.configure do |c|
  c.before :all do
    c.os = backend(Serverspec::Commands::Base).check_os
  end
end
{% endcodeblock %}

This code is modeled after the examples provided on the
[serverspec](http://serverspec.org/tutorial.html) website.

Create a subdirectory underneath `myface/test/integration/default/serverspec`
called `localhost`:

    myface/test/integration/default/serverspec/localhost

It is a serverspec convention to put tests (a.k.a. "specs") underneath
`spec_helper.rb` in a subdirectory denoting the host name to be tested.
Serverspec supports testing via SSH access to remote hosts.  We won't
be using this capability as we will be testing local images, so we'll just
use `localhost` for the host name.

Now, let's write our first test!  If you recall in 
[Testing Iteration #1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/#testing-iteration-1), we ran
the following command to verify that the myface user was created:

    $ getent password myface
    myface:x:497:503::/home/myface:/bin/bash

Create the a file named `myface/test/integration/default/serverspec/localhost/webserver_spec.rb` that contains a serverspec test to perform the same action:

{% codeblock myface/test/integration/default/serverspec/localhost/webserver_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace webserver' do

  it 'should have a myface user' do
    expect(command 'getent passwd myface').to return_stdout /myface:x:\d+:\d+::\/home\/myface:\/bin\/bash/
  end

end
{% endcodeblock %}

Serverspec provides extensions to Rspec to help you more easily test servers.
If you're not familiar with Rspec syntax, Code School has an excellent
tutorial on [Testing with Rspec](http://rspec.codeschool.com/).  Even if you
don't know Rspec, you should still be able to follow along with
provided examples.

You can find a list of serverspec resources at the following link:
<http://serverspec.org/resource_types.html>.  We're using the `command`
resource to run the command `getent password myface` and match the
resultant output with a Ruby regular expression (because the uid and gid
field could be any number between 100-999, because myface is a system
account).


Testing Iteration #16 - Running your first Serverspec test
----------------------------------------------------------

OK, let's run our first test!

To start you need to run `kitchen setup` so that Test Kitchen loads and
configures all the required plugins.  In keeping with the restaurant theme,
the component that manages Test Kitchen plugins is called
[Busser](http://en.wikipedia.org/wiki/Busser).

    $ kitchen setup
    -----> Starting Kitchen (v1.1.1)
    -----> Setting up <default-centos65>...
    Fetching: thor-0.18.1.gem (100%)
    Fetching: busser-0.6.0.gem (100%)
           Successfully installed thor-0.18.1
           Successfully installed busser-0.6.0
           2 gems installed
    -----> Setting up Busser
           Creating BUSSER_ROOT in /tmp/busser
           Creating busser binstub
           Plugin serverspec installed (version 0.2.5)
    -----> Running postinstall for serverspec plugin
           Finished setting up <default-centos65> (0m52.12s).
    -----> Setting up <default-ubuntu1204>...
    Fetching: thor-0.18.1.gem (100%)
    Fetching: busser-0.6.0.gem (100%)
    Successfully installed thor-0.18.1
    Successfully installed busser-0.6.0
    2 gems installed
    -----> Setting up Busser
           Creating BUSSER_ROOT in /tmp/busser
           Creating busser binstub
           Plugin serverspec installed (version 0.2.5)
    -----> Running postinstall for serverspec plugin
           Finished setting up <default-ubuntu1204> (0m15.36s).
    -----> Kitchen is finished. (1m7.77s)

After running `kitchen setup`, next run `kitchen verify` to run your test
suite.

    $ kitchen verify
    -----> Starting Kitchen (v1.1.1)
    -----> Verifying <default-centos65>...
           Suite path directory /tmp/busser/suites does not exist, skipping.
           Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
           Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
           /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

           MyFace webserver
             should have a myface user

           Finished in 0.04293 seconds
           1 example, 0 failures
           Finished verifying <default-centos65> (0m1.61s).
    -----> Verifying <default-ubuntu1204>...
           Suite path directory /tmp/busser/suites does not exist, skipping.
    Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
    Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
     /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

    MyFace webserver
      should have a myface user

    Finished in 0.11307 seconds
    1 example, 0 failures
           Finished verifying <default-ubuntu1204> (0m1.66s).
    -----> Kitchen is finished. (0m3.80s)

Finally run `kitchen list` to display the results of your test run.

    $ kitchen list
    Instance             Driver   Provisioner  Last Action
    default-centos-64    Vagrant  Chef Solo    Verified
    default-ubuntu-1204  Vagrant  Chef Solo    Verified

If Test Kitchen displays the `Last Action` as `Verified`, all the tests passed.

Iteration #17 - Completing the webserver test suite
===================================================

Now let's dive in and encode all the rest of the tests from
[Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/).

While we used the `command` resource to encode our first test, this isn't
the optimal way to encode this test as a serverspec.  We can make use of the 
`user` resource to encode a test more succinctly:

    it 'should have a myface user' do
      expect(user 'myface').to exist
    end

The `myface/test/integration/default/serverspec/localhost/webserver_spec.rb`
file should resemble the following:

{% codeblock myface/test/integration/default/serverspec/localhost/webserver_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace webserver' do

  it 'should have a myface user' do
    expect(user 'myface').to exist
  end

end
{% endcodeblock %}

Run `kitchen verify` and `kitchen list` to re-run your test.  You should see
the same result as before - `1 example, 0 failures`:

    $ kitchen verify
    $ kitchen list

Only use the `command` resource as a method of last resort.  First check to
see if serverspec has a better resource to perform a test.

Let's move on to the next command from
[Testing Iteration #3](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/#testing-iteration-3):

    $ vagrant ssh -c "sudo /sbin/service httpd status"
    httpd (pid  4831) is running.

Use the servspec `service` resource to perform a test to ensure that the
httpd service is running and it automatically starts on bootup:

    it 'should be running the httpd server' do
      expect(service 'httpd').to be_running
      expect(service 'httpd').to be_enabled
    end

Add this statement to your `webserver_spec` file:

{% codeblock myface/test/integration/default/serverspec/localhost/webserver_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace webserver' do

  it 'should have a myface user' do
    expect(user 'myface').to exist
  end

  it 'should be running the httpd server' do
    expect(service 'httpd').to be_running
    expect(service 'httpd').to be_enabled
  end

end
{% endcodeblock %}

Run `kitchen verify` and `kitchen list` again to run this new test:

    $ kitchen verify
    -----> Starting Kitchen (v1.1.1)
    -----> Verifying <default-centos65>...
    ...
           MyFace webserver
             should have a myface user
             should be running the httpd server

           Finished in 0.04706 seconds
           2 examples, 0 failures
           Finished verifying <default-centos65> (0m1.61s).
    -----> Verifying <default-ubuntu1204>...
           Removing /tmp/busser/suites/serverspec
    Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
    Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
    /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

    MyFace webserver
      should have a myface user
    httpd: unrecognized service
      should be running the httpd server (FAILED - 1)

    Failures:

      1) MyFace webserver should be running the httpd server
         Failure/Error: expect(service 'httpd').to be_running
           ps aux | grep -w -- httpd | grep -qv grep
           expected Service "httpd" to be running
         # /tmp/busser/suites/serverspec/localhost/webserver_spec.rb:10:in `block (2 levels) in <top (required)>'

    Finished in 0.05906 seconds
    2 examples, 1 failure

    Failed examples:

    rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb:9 # MyFace webserver should be running the httpd server
    /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation failed
    Ruby Script[/tmp/busser/gems/gems/busser-serverspec-0.2.5/lib/busser/runner_plugin/../serverspec/runner.rb /tmp/busser/suites/serverspec] exit code was 1
    >>>>>> Verify failed on instance <default-ubuntu1204>.
    >>>>>> Please see .kitchen/logs/default-ubuntu1204.log for more details
    >>>>>> ------Exception-------
    >>>>>> Class: Kitchen::ActionFailed
    >>>>>> Message: SSH exited (1) for command: [sh -c 'BUSSER_ROOT="/tmp/busser" GEM_HOME="/tmp/busser/gems" GEM_PATH="/tmp/busser/gems" GEM_CACHE="/tmp/busser/gems/cache" ; export BUSSER_ROOT GEM_HOME GEM_PATH GEM_CACHE; sudo -E /tmp/busser/bin/busser test']
    >>>>>> ----------------------
    ...
    $ kitchen list
    Instance            Driver   Provisioner  Last Action
    default-centos65    Vagrant  ChefSolo     Verified
    default-ubuntu1204  Vagrant  ChefSolo     Verified

Uh oh!  That's not what we expected! The tests failed on our Ubuntu 12.04
instance - and yet it still says that it is `Verified`, but the tests passed
on CentOS 6.5.  The `Last Action` field is literally the last action.  It does
not report success or failure state, so you'll want to pay attention to the
output of `kitchen verify` and note whether or not all the tests passed.

In this case, the reason for the failure is that on Ubuntu, the name of the
Apache httpd service is `apache2` not `httpd`.  Let's address this by adding a conditional that checks the `os` custom configuration setting that is set in 
`spec_helper.rb`.

I didn't explain what this did before, but it runs a serverspec helper
method to check the os type before each spec/test run.  When running under
Ubuntu (or Debian), the value of `RSpec.configuation.os` will be `Debian`,
otherwise the value will be `RedHat` if it is running under any RHEL variant,
including CentOS.  So the following conditional should do the trick:

    it 'should be running the httpd server' do
      case RSpec.configuration.os
      when "Debian"
        expect(service 'apache2').to be_running
        expect(service 'apache2').to be_enabled
      else
        expect(service 'httpd').to be_running
        expect(service 'httpd').to be_enabled
      end
    end 

After this change, your `webserver_spec.rb` file should resemble the following:
 
{% codeblock myface/test/integration/default/serverspec/localhost/webserver_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace webserver' do

  it 'should have a myface user' do
    expect(user 'myface').to exist
  end

  it 'should be running the httpd server' do
    case RSpec.configuration.os[:family]
    when "Ubuntu"
      expect(service 'apache2').to be_running
      expect(service 'apache2').to be_enabled
    else
      expect(service 'httpd').to be_running
      expect(service 'httpd').to be_enabled
    end
  end

end
{% endcodeblock %}

Run `kitchen verify` and `kitchen list` again - all the tests should pass:

    $ kitchen verify
    -----> Starting Kitchen (v1.1.1)
    -----> Verifying <default-centos65>...
           Removing /tmp/busser/suites/serverspec
           Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
           Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
           /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

           MyFace webserver
             should have a myface user
             should be running the httpd server

           Finished in 0.04858 seconds
           2 examples, 0 failures
           Finished verifying <default-centos65> (0m1.58s).
    -----> Verifying <default-ubuntu1204>...
           Removing /tmp/busser/suites/serverspec
    Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
    Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
    /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

    MyFace webserver
      should have a myface user
      should be running the httpd server

    Finished in 0.06551 seconds
    2 examples, 0 failures
           Finished verifying <default-ubuntu1204> (0m1.59s).
    -----> Kitchen is finished. (0m3.46s)

    $ kitchen list
    Instance             Driver   Provisioner  Last Action
    default-centos65     Vagrant  Chef Solo    Verified
    default-ubuntu1204   Vagrant  Chef Solo    Verified

The final test that is used for the rest of the Test Iterations in
[Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)
basically amounts to visiting http://33.33.33.10 with a web browser
and eyeballing the results.  That would be difficult to automate with
serverspec, and one would probably want to use a web automation framework
like [Selenium](http://docs.seleniumhq.org/) to do this.  However, you can
at least use serverspec to verify that the Apache Server is serving up
content on port 80 (the default http port).

We can check that the server is listening on port 80 with the `port` resource:

    it 'should be listening on port 80' do
      expect(port 80).to be_listening
    end

We'll resort to using the `command` resource to check to see if the server
accepts an HTTP connections and returns something that looks reasonable, as
there doesn't seem to be an obvious higher-level resource to perform this
action:

    it 'should respond to an HTTP request' do
      expect(command 'curl localhost').to return_stdout /.*<title>MyFace Users<\/title>.*/
    end

After adding these two checks, this is what your `webserver_spec.rb` file should
look like:

{% codeblock myface/test/integration/default/serverspec/localhost/webserver_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace webserver' do

  it 'should have a myface user' do
    expect(user 'myface').to exist
  end

  it 'should be running the httpd server' do
    case RSpec.configuration.os[:family]
    when "Ubuntu"
      expect(service 'apache2').to be_running
      expect(service 'apache2').to be_enabled
    else
      expect(service 'httpd').to be_running
      expect(service 'httpd').to be_enabled
    end
  end

  it 'should respond to an HTTP request' do
    expect(command 'curl localhost').to return_stdout /.*<title>MyFace Users<\/title>.*/
  end

end
{% endcodeblock %}

Now we have an automated script that performs some basic tests to verify
that our cookbook enabled the web server properly.  Let the robots do some
of the grunge work!

Testing Iteration #17 - Running the suite
-----------------------------------------

Do a final `kitchen verify` and `kitchen list`.  Everything should look good:

    $ kitchen verify
    -----> Starting Kitchen (v1.1.1)
    -----> Verifying <default-centos65>...
           Removing /tmp/busser/suites/serverspec
           Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
           Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
           /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

           MyFace webserver
             should have a myface user
             should be running the httpd server
             should respond to an HTTP request

           Finished in 0.05525 seconds
           3 examples, 0 failures
           Finished verifying <default-centos65> (0m1.53s).
    -----> Verifying <default-ubuntu1204>...
           Removing /tmp/busser/suites/serverspec
    Uploading /tmp/busser/suites/serverspec/localhost/webserver_spec.rb (mode=0644)
    Uploading /tmp/busser/suites/serverspec/spec_helper.rb (mode=0644)
    -----> Running serverspec test suite
    /opt/chef/embedded/bin/ruby -I/tmp/busser/suites/serverspec -S /opt/chef/embedded/bin/rspec /tmp/busser/suites/serverspec/localhost/webserver_spec.rb --color --format documentation

    MyFace webserver
      should have a myface user
      should be running the httpd server
      should respond to an HTTP request

    Finished in 0.15965 seconds
    3 examples, 0 failures
           Finished verifying <default-ubuntu1204> (0m1.68s).
    -----> Kitchen is finished. (0m3.53s)

    $ kitchen list
    Instance             Driver   Provisioner  Last Action
    default-centos65     Vagrant  Chef Solo    Verified
    default-ubuntu1204   Vagrant  Chef Solo    Verified

Iteration #18 - Completing the database test suite
==================================================

Let's wrap this up by finishing off the tests for the database portion in
[Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/)
Create a new file called `myface/test/integration/default/serverspec/localhost/database_spec.rb` to contain the database tests.

In [Testing Iteration #7](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/#testing-iteration-7)
we checked to see if the `mysqld` service was running with the following
command:

    $ sudo /sbin/service mysqld status

There is a similar name difference between the Ubuntu and CentOS services as
there was with the Apache web server.  For Ubuntu, the name of the
MySQL service is `mysql`.  For CentOS, the name of the service is `mysqld`.

This should be a piece of cake to write a serverspec test for now:

    it 'should be running the database server' do
      case RSpec.configuration.os[:family]
      when "Ubuntu"
        expect(service 'mysql').to be_running
        expect(service 'mysql').to be_enabled
      else
        expect(service 'mysqld').to be_running
        expect(service 'mysqld').to be_enabled
      end
    end

In [Testing Iteration #8](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/#testing-iteration-8)
we ran the following command to verify that the myface database was created:

    $ mysqlshow -uroot -prootpass

That's a simple `command` resource regular expression:

    it 'should have created the myface database' do
      expect(command 'mysqlshow -uroot -prootpass').to return_stdout /.*myface.*/
    end

In [Testing Iteration #9](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/#testing-iteration-9)
we created a myface-app MySQL database user and to check to see if the
myface_app user only has rights to the myface database with the following
commands:

    $ mysql -uroot -prootpass -e "select user,host from mysql.user;"
    $ mysql -uroot -prootpass -e "show grants for 'myface_app'@'localhost';"

Again, these are just more serverspec `command`s (`\s` indicates "any
whitespace character"):

    it 'should have created the myface_app user' do
      expect(command 'mysql -uroot -prootpass -e "select user,host from mysql.user;"').to return_stdout /.*myface_app\s+localhost.*/
    end

    it 'should have given the myface_app database user rights to myface' do
      expect(command 'mysql -uroot -prootpass -e "show grants for \'myface_app\'@\'localhost\';"').to return_stdout /.*GRANT ALL PRIVILEGES ON `myface`.\* TO \'myface_app\'@\'localhost\'.*/
    end

In [Testing Itegration #10](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/#testing-iteration-10)
we dumped the contents of the `users` table to verify it got created:

    $ mysql -hlocalhost -umyface_app -psupersecret -Dmyface -e "select id,user_name from users;"'

You guessed it, yet another `command`:

    it 'should have created the users table' do
      expect(command 'mysql -hlocalhost -umyface_app -psupersecret -Dmyface -e "select id,user_name from users;"').to return_stdout /.*mbower.*/
    end

In [Testing Iteration #11](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/#test-iteration-11) we checked
to see if the php5_module was successfully installed:

    $ sudo /usr/sbin/httpd -M | grep php5

Note that there is a `php_config` serverspec resource for checking
PHP config settings, but that's not helpful for checking the existence
of PHP, so another command will do (just remember the service name is
different between the two different OSes):

    it 'should have installed the Apache php5_module' do
      case RSpec.configuration.os[:family]
      when "Ubuntu"
        expect(command 'sudo /usr/sbin/apache2 -M | grep php5').to return_stdout /.*php5_module.*/
      else
        expect(command 'sudo /usr/sbin/httpd -M | grep php5').to return_stdout /.*php5_module.*/
      end
    end

And there you have it!  Your final `database_spec.rb` file should resemble the
following:

{% codeblock myface/test/integration/default/serverspec/localhost/database_spec.rb lang:ruby %}
require 'spec_helper'

describe 'MyFace database' do

  it 'should be running the database server' do
    case RSpec.configuration.os[:family]
    when "Ubuntu"
      expect(service 'mysql').to be_running
      expect(service 'mysql').to be_enabled
    else
      expect(service 'mysqld').to be_running
      expect(service 'mysqld').to be_enabled
    end
  end

  it 'should have created the myface database' do
    expect(command 'mysqlshow -uroot -prootpass').to return_stdout /.*myface.*/
  end

  it 'should have created the myface_app user' do
    expect(command 'mysql -uroot -prootpass -e "select user,host from mysql.user;"').to return_stdout /.*myface_app\s+localhost.*/
  end

  it 'should have given the myface_app database user rights to myface' do
    expect(command 'mysql -uroot -prootpass -e "show grants for \'myface_app\'@\'localhost\';"').to return_stdout /.*GRANT ALL PRIVILEGES ON `myface`.\* TO \'myface_app\'@\'localhost\'.*/
  end

  it 'should have created the users table' do
    expect(command 'mysql -hlocalhost -umyface_app -psupersecret -Dmyface -e "select id,user_name from users;"').to return_stdout /.*mbower.*/
  end

  it 'should have installed the Apache php5_module' do
    case RSpec.configuration.os[:family]
    when "Ubuntu"
      expect(command 'sudo /usr/sbin/apache2 -M | grep php5').to return_stdout /.*php5_module.*/
    else
      expect(command 'sudo /usr/sbin/httpd -M | grep php5').to return_stdout /.*php5_module.*/
    end
  end

end
{% endcodeblock %}

Testing Iteration #19 - kitchen test
------------------------------------

Perform a final `kitchen verify` and `kitchen list` to check that there are
no syntax errors.  9 tests succeeded!

In addition to the `kitchen` commands that you have used so far, there's one
other command that it quite useful - `kitchen test`.  It runs all the commands
in the Test Kitchen test lifecycle in order:

`kitchen create` - Creates a vagrant instance.

`kitchen converge` - Provision the vagrant instance with Chef, using the run list specified in the `.kitchen.yml` file.

`kitchen setup` - Install and configure any necessary Test Kitchen plugins needed to run tests.

`kitchen verify` - Run tests.

`kitchen destroy` - Destroy the vagrant instance, removing it from memory & disk.

When you are in the midst of writing tests, using the above commands
interactively can save time (like only running `kitchen verify` after adding
a new test).  But once the tests are written, normally you will run
`kitchen test` to run everything in one shot, preferably running as a "latch"
triggered when your cookbook changes are committed to source control.  This
will ensure that your tests are run often.

Conclusion
==========

So hopefully now you understand how to use Test Kitchen and what it's useful
for.  In the next article in this series, we'll cover writing tests that can
run before deployment, providing feedback more quickly than with
Test Kitchen (albeit in more limited circumstances), using Chefspec and Guard.

Now you should be able to test everything but the kitchen sink!  Wait...
actually Test Kitchen has got that covered as well:

    $ kitchen sink

                         ___
                        ' _ '.
                      / /` `\ \
                     | |   [__]
                     | |    \{\{
                     | |    \}\}
                  _  | |  _ \{\{
      ___________<_>_| |_<_>\}\}________
          .=======^=(___)=^=\{\{====.
         / .----------------\}\}---. \
        / /                 \{\{    \ \
       / /                  \}\}     \ \
      (  '========================='  )
       '-----------------------------'
