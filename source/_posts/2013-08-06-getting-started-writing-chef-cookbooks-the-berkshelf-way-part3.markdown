---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 3"
date: 2013-08-06 12:46
comments: true
categories: chef
---
* list element with functor item
{:toc}

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
    box: opscode-centos-6.4
    box_url: https://opscode-vm-bento.s3.amazonaws.com/vagrant/opscode_centos-6.4_provisionerless.box

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

Testing Iteration #14 - Provision with Test Kitchen
---------------------------------------------------

You can do everything that you were doing with vagrant just using Test Kitchen.
The Test Kitchen equivalent of the `vagrant up` command is `kitchen setup`.
Try running the `kitchen setup` command now to verify that your
`.kitchen.yml` file is valid.  When you run `kitchen setup` it will
spin up a CentOS 6.4 vagrant test node instance and use Chef Solo to provision
the MyFace cookbook on the test node:

    $ kitchen setup
    -----> Starting Kitchen (v1.0.0.beta.2)
    -----> Creating <default-centos-64>
           [kitchen::driver::vagrant command] BEGIN (vagrant up --no-provision)
           Bringing machine 'default' up with 'virtualbox' provider...
           [default] Importing base box 'opscode-centos-6.4'...
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
