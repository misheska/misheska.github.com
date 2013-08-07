---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way"
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
Edit the `myface/metadata.rb` file and all the following line to load the
Test Kitchen gem:

    gem 'test-kitchen', '~> 1.0.0.beta.2'

Your `myface/metadata.rb` file should look like the following:

{% codeblock myface/metadata.rb lang:ruby %}
source 'https://ruby

    gem 'berkshelf'
    gem 'test-kitchen', '~> 1.0.0.beta.2'
{% endcodeblock %}

Test Kitchen is still in beta release and is still evolving rapidly.  The
1.0.0.beta.2 version constraint will ensure that you are using the latest
stable prerelease version:

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

Verifying Iteration #13 - Show the Test Kitchen version
-------------------------------------------------------

If everything worked properly you should be able to run the `kitchen --version`
command to see your installed Test Kitchen's version information

    $ kitchen --version
    Test Kitchen version 1.0.0.beta.2

Iteration #14 - Create a Kitchen YAML file
==========================================
In order to use Test Kitchen on a cookbook, first you need to add a few more
dependencies and create a template Kitchen YAML file.  Test Kitchen makes this
easy by providing a command to do all the initialization steps automatically 

    $ kitchen init
          create  .kitchen.yml
          append  Thorfile
          create  test/integration/default
          append  .gitignore
          append  .gitignore
          append  Gemfile
    You must run `bundle install' to fetch any new gems.
