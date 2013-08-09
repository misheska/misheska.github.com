---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 3"
date: 2013-08-06 12:46
comments: true
categories: chef
---
* list element with functor item
{:toc}

NOTE: Preview for beta testers to review...

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
Edit `myface/Gemfile` and add the following line to load the
Test Kitchen gem:

    gem 'test-kitchen', '~> 1.0.0.beta.2'

Your `myface/Gemfile` should look like the following after editing:

{% codeblock myface/Gemfile lang:ruby %}
source 'https://rubygems.org'

gem 'berkshelf'
gem 'test-kitchen', '~> 1.0.0.beta.2'
{% endcodeblock %}

Test Kitchen is in beta release and is still evolving rapidly.  The
1.0.0.beta.2 version constraint will ensure that you are using the latest
stable prerelease version.

After you have updated the `Gemfile` run `bundle install` to download the
test-kitchen gem and all its dependencies:

    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.4)
    Using multi_json (1.7.8)
    Using activesupport (3.2.14)
    ...
    Installing test-kitchen (1.0.0.beta.2)
    Using bundler (1.3.5)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

Testing Iteration #13 - Show the Test Kitchen version
-----------------------------------------------------

If everything worked properly you should be able to run the `kitchen --version`
command to see your installed Test Kitchen's version information

    $ kitchen --version
    Test Kitchen version 1.0.0.beta.2

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
    You must run 'bundle install' to fetch any new gems.

Since `kitchen init` modified your Gemfile, you need to re-run `bundle install`
(as suggested above) to pick up the new gem dependencies:

    $ bundle install
    Fetching gem metadata from https://rubygems.org/........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Using i18n (0.6.4)
    Using multi_json (1.7.8)
    Using activesupport (3.2.14)
    ...
    Using safe_yaml (0.9.5)
    Using test-kitchen (1.0.0.beta.2)
    Installing kitchen-vagrant (0.11.0)
    Using bundler (1.3.5)
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
driver_plugin: vagrant
driver_config:
  require_chef_omnibus: true

platforms:
- name: centos-6.4
  driver_config:
    box: misheska-centos64
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-centos64.box

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes: { mysql: { server_root_password: "rootpass", server_debian_password: "debpass", server_repl_password: "replpass" } }

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
driver_plugin: vagrant
driver_config:
  require_chef_omnibus: true

platforms:
- name: centos-6.4
  driver_config:
    box: misheska-centos64
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-centos64.box
    network:
    - ["private_network", {ip: "33.33.33.10"}]

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes: { mysql: { server_root_password: "rootpass", server_debian_password: "debpass", server_repl_password: "replpass" } }

{% endcodeblock %}

For more information on the `kitchen-vagrant` settings, refer to the
README.md file for `kitchen-vagrant` at <https://github.com/opscode/kitchen-vagrant/blob/master/README.md>

Testing Iteration #14 - Provision with Test Kitchen
---------------------------------------------------

You can do nearly everything that you were doing with vagrant just using Test
Kitchen.  The Test Kitchen equivalent of the `vagrant up` command is 
`kitchen setup`.  Try running the `kitchen setup` command now to verify that
your `.kitchen.yml` file is valid.  When you run `kitchen setup` it will
spin up a CentOS 6.4 vagrant test node instance and use Chef Solo to provision
the MyFace cookbook on the test node:

    $ kitchen setup
    -----> Starting Kitchen (v1.0.0.beta.2)
    -----> Creating <default-centos-64>
           [kitchen::driver::vagrant command] BEGIN (vagrant up --no-provision)
           Bringing machine 'default' up with 'virtualbox' provider...
           [default] Importing base box 'misheska-centos64'...
           [default] Matching MAC address for NAT networking...
           [default] Setting the name of the VM...
           [default] Clearing any previously set forwarded ports...
           [Berkshelf] Skipping Berkshelf with --no-provision
           [default] Fixed port collision for 22 => 2222. Now on port 2200.
           [default] Creating shared folders metadata...
           [default] Clearing any previously set network interfaces...
           [default] Preparing network interfaces based on configuration...
           [default] Forwarding ports...
           [default] -- 22 => 2200 (adapter 1)
           [default] Running 'pre-boot' VM customizations...
           [default] Booting VM...
           [default] Waiting for VM to boot. This can take a few minutes.
           [default] VM booted and ready for use!
           [default] Setting hostname...
           [default] Mounting shared folders...
           [default] -- /vagrant
           [kitchen::driver::vagrant command] END (0m39.38s)
           [kitchen::driver::vagrant command] BEGIN (vagrant ssh-config)
           [kitchen::driver::vagrant command] END (0m2.85s)
           Vagrant instance <default-centos-64> created.
           Finished creating <default-centos-64> (0m46.19s).
    -----> Converging <default-centos-64>
    -----> Installing Chef Omnibus (true)
           --2013-08-07 07:25:35--  https://www.opscode.com/chef/install.sh
    Resolving www.opscode.com...        184.106.28.82
           Connecting to www.opscode.com|184.106.28.82|:443...
           connected.
    HTTP request sent, awaiting response...        200 OK
           Length: 6790 (6.6K) [application/x-sh]
           Saving to: “STDOUT”
    
     0% [                                       ] 0           --.-K/s
    100%[======================================>] 6,790       --.-K/s   in 0s
     
           2013-08-07 07:25:40 (461 MB/s) - written to stdout [6790/6790]
    
           Downloading Chef  for el...
           Installing Chef
           warning: /tmp/tmp.wAJrUjYo/chef-.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
    Preparing...                #####  ########################################### [100%]
       1:chef                          ########################################### [100%]
           Thank you for installing Chef!
           Resolving cookbook dependencies with Berkshelf
    Using myface (2.0.0)
    ...
           Recipe: apache2::default
             * service[apache2] action restart[2013-08-07T07:28:54+00:00] INFO: Processing service[apache2] action restart (apache2::default line 221)
           [2013-08-07T07:28:55+00:00] INFO: service[apache2] restarted
    
               - restart service service[apache2]
    
           [2013-08-07T07:28:55+00:00] INFO: Chef Run complete in 162.083058446 seconds
           [2013-08-07T07:28:55+00:00] INFO: Running report handlers
           [2013-08-07T07:28:55+00:00] INFO: Report handlers complete
           Chef Client finished, 97 resources updated
           Finished converging <default-centos-64> (3m20.62s).
    -----> Setting up <default-centos-64>
           Finished setting up <default-centos-64> (0m0.00s).
    -----> Kitchen is finished. (4m11.93s)

To display the results of the Chef Run, type in the `kitchen list` command:

    $ kitchen list
    Instance           Driver   Provisioner  Last Action
    default-centos-64  Vagrant  Chef Solo    Set Up

If the run succeeded, it should display `Set Up` in the `Last Action` field.

The Test Kitchen equivalent of the `vagrant ssh` command is `kitchen login`.
Since Test Kitchen supports multiple instances, you will need to pass in
the instance name for which you wish to login as a parameter (which you can
get from the `kitchen list` output).  We want to login to the CentOS 6.4
instance (the only instance for now), so type in the command
`kitchen login default-centos-64` to login:

    $ kitchen login default-centos-64
    Last login: Wed Aug  7 07:39:31 2013 from 10.0.2.2
    [vagrant@default-centos-64 ~]$

Now you can poke around in the image the same way you did with `vagrant ssh`,
for example, verifying that the httpd server is running:

    [vagrant@default-centos-64 ~]$ sudo /sbin/service httpd status
    httpd (pid  3737) is running...

After you are done working in the test instance, make sure to run the
`exit` command to log out so that you return back to your host prompt:

    [vagrant@default-centos-64 ~]$ exit
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

Iteration #15 - Provisioning on Ubuntu
======================================

We haven't really made use of any unique Test Kitchen features yet, let's
start now.  We'll also deploy our cookbook locally to Ubuntu 12.04 for
testing, in addition to CentOS 6.4.

Edit `.kitchen.yml` and add a reference to a Ubuntu 12.04 basebox alongside
the existing CentOS 6.4 basebox in the `platforms` stanza:

    -name: ubuntu-12.04
     driver_config:
       box: misheska-ubuntu1204
       box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-ubuntu1204.box
       network:
       - ["private_network", {ip: "33.33.33.11"}]

{% codeblock myface/.kitchen.yml %}

---
driver_plugin: vagrant
driver_config:
  require_chef_omnibus: true

platforms:
- name: centos-6.4
  driver_config:
    box: misheska-centos64
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-centos64.box
    network:
    - ["private_network", {ip: "33.33.33.10"}]
- name: ubuntu-12.04
  driver_config:
    box: misheska-ubuntu1204
    box_url: https://s3-us-west-2.amazonaws.com/misheska/vagrant/virtualbox/misheska-ubuntu1204.box
    network:
    - ["private_network", {ip: "33.33.33.11"}]

suites:
- name: default
  run_list: ["recipe[myface]"]
  attributes: { mysql: { server_root_password: "rootpass", server_debian_password: "debpass", server_repl_password: "replpass" } }

{% endcodeblock %}

Before we run `kitchen setup` to do a Chef run, we need to fix our cookbook
so it will run successfully on Ubuntu 12.04.  If you tried to deploy now you
would notice that the MyFace cookbook would fail to deploy to Ubuntu 12.04
successfully due to a reference to the `php-mysql` package in 
`myface/receipes/webserver.rb`.

    ... 
    include_recipe "apache2"
    include_recipe "apache2::mod_php5"

    package "php-mysql" do
      action :install
      notifies :restart, "service[apache2]"
    end
    ...

On Ubuntu the package name should be `php5-mysql` instead of `php-mysql`.

As with most issues in the Chef world, there's a cookbook for that!
The Opscode `php` cookbook has conditionals to reference the correct name
for the `php-mysql` package on a number of platforms.

Edit `myface/metadata.rb` and add a reference to the latest version of the
`php` cookbook (currently 1.2.3):

{% codeblock myface/metadata.rb %}
name             'myface'
maintainer       'Mischa Taylor'
maintainer_email 'mischa@misheska.com'
license          'Apache 2.0'
description      'Installs/Configures myface'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '2.0.0'

depends "apache2", "~> 1.6.0"
depends "mysql", "~> 3.0.0"
depends "database", "~> 1.3.0"
depends "php", "~> 1.2.0"
{% endcodeblock %}

In `myface/recipes/webserver.rb` replace the `package "php-mysql" do ... end`
block with the following reference:

    include_recipe "php::module_mysql"

After editing, `myface/recipes/webserver.rb` should look like this:

{% codeblock myface/recipes/metadata.rb %}
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
include_recipe "apache2::mod_php5"

include_recipe "php::module_mysql"

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
  mode "0755"
  recursive true
end

# write site
template "#{node[:myface][:document_root]}/index.php" do
  source "index.php.erb"
  mode "0644"
end

# enable myface
apache_site "#{node[:myface][:config]}" do
  enable true
end
{% endcodeblock %}

Testing Iteration #15 - Deploy locally to Ubuntu 12.04
------------------------------------------------------

Now that we've fixed up our cookbook to work on Ubuntu 12.04, let's test it
out!  Run `kitchen list` to display the list of Test Kitchen instances:

    Instance             Driver   Provisioner  Last Action
    default-centos-64    Vagrant  Chef Solo    Set Up
    default-ubuntu-1204  Vagrant  Chef Solo    <Not Created>

Notice that after editing the `.kitchen.yml` we now have an Ubuntu 12.04
instance called `default-ubuntu-1204` and it is in the `<Not Created>`
state.

Go ahead setup the Ubuntu 12.04 instance by running the following command:

    $ kitchen setup default-ubuntu-1204

Note that this time we added an optional instance parameter so that Test
Kitchen only performs the action against the specified instance.  If you do
not specify this parameter, it defaults to `all`, running the command against
all instances.  After about 5-10 minutes or so, you should observe that Test
Kitchen downloaded an Ubuntu 12.04 basebox, booted a VM with the basebox, and
successfully deployed our chef cookbook.

Run the `kitchen list` command again to verify that the Ubuntu 12.04 instance
is now in the `Set Up` state as well, showing that there were no errors:

    Instance             Driver   Provisioner  Last Action
    default-centos-64    Vagrant  Chef Solo    Set Up
    default-ubuntu-1204  Vagrant  Chef Solo    Set Up

You just fixed an error with the MyFace cookbook that prevented deployment to
Ubuntu 12.04, and verified that the cookbook correctly deploys to both 
Ubuntu 12.04 and Centos 6.4.

Use the `kitchen login` command to ssh into each instance and poke around
if you like.  You now have two local vagrant VMs instantiated to play with!

    $ kitchen login default-ubuntu-1204
    Welcome to Ubuntu 12.04.2 LTS (GNU/Linux 3.5.0-23-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com/
    Last login: Thu Aug  8 08:40:50 2013 from 10.0.2.2
    $ [...poke around, run some commands...]
    $ exit
    Connection to 127.0.0.1 closed.

    $ kitchen login default-centos-64
    Last login: Thu Aug  8 08:54:44 2013 from 10.0.2.2
    Welcome to your Packer-built virtual machine.
    [vagrant@default-centos-64 ~]$ [...poke around, run some commands...]
    [vagrant@default-centos-64 ~]$ exit
    logout
    Connection to 127.0.0.1 closed.

You can view the websites for each instance by viewing the appropriate
private IP

    CentOS 6.4:   http://33.33.33.10
    Ubuntu 12.04: http://33.33.33.11
