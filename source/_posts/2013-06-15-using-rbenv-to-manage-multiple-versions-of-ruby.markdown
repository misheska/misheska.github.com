---
layout: post
title: "Using Rbenv to manage multiple versions of Ruby"
date: 2013-06-15 15:53
comments: true
categories: ruby
---
* list element with functor item
{:toc}

Out of the box, Ruby does not provide a mechanism to support multiple
installed versions.  Compounding this issue, the default system-installed
version of Ruby for most versions of Linux/Mac OS X tend to be quite old.
For example, even in the latest version of Mac OS X Mountain Lion, the
system-wide version of Ruby is over five years old (ruby 1.8.7).  Most
developers prefer to use a newer version of Ruby installed in their home
directory and to leave the default systemwide version of Ruby untouched.

[Rbenv](https://github.com/sstephenson/rbenv/) makes managing multiple
versions of Ruby easy.  It's a great way to work on current development
projects using Ruby 1.9.x and be able to switch to Ruby 2.0.x for new
work.  Rbenv is a lightweight alternative to
[Ruby Version Manager (RVM)](http://rvm.io).  Rbenv does not include
any mechanism to install Ruby or manage gems, like with RVM.

Rbenv is supported on Linux and Mac OS X.  Rbenv doesn't work on Windows.
For Windows, [Pik](https://github.com/vertiginous/pik) is an Rbenv alternative.

NOTE: Rbenv is incompatible with RVM because RVM overrides the
`gem` command with a function.  If you want to switch to Rbenv,
make sure you uninstall RVM first.  Run the following commands to uninstall
RVM:

    $ rvm implode
    $ gem uninstall rvm

Then remove/comment out the RVM loader line in `.bash_profile`
and/or `.bashrc`

Mac OS X
========

Install Rbenv and Ruby 1.9.x - Mac OS X
---------------------------------------
First you'll need to install a C compiler to build Ruby from source.  Just
download and install the latest version of Xcode from the App Store, if you
don't have it installed already.  Also make sure you install the *Command Line
Tools* by choosing the menu item <code>Xcode -> Preferences...</code> and click
on the *Downloads* tab.  Click on the *Install* button to download the
Command Line Tools.
![Xcode Command Line Tools](/images/xcodecommandline.png)

Next, you'll need to install the Homebrew package manager to get all the
dependencies needed to compile Ruby from source.  While you could manage
these dependencies by hand, using a package manager is a better idea, as
package managers know how to uninstall what they install.

Run the following command to install Homebrew:

    $ ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"

Run `brew doctor` and address any issues it discovers.  When
all is well, you should see:

    $ brew doctor
    Your system is raring to brew.

Next, install the additional dependencies to compile Ruby from source:

    # For update-system
    brew update
    brew tap homebrew/dupes
    # For ruby
    brew install apple-gcc42

And now install `rbenv` and the `ruby-build` plugin:

    brew update
    brew install rbenv
    brew install ruby-build

Add <code>rbenv init</code> to your shell to enable shims and autocompletion:

    $ echo 'eval "$(rbenv init -)"' >> $HOME/.bash_profile
    $ source ~/.bash_profile

Restart shell as a login shell so that the PATH changes take effect:

    $ exec $SHELL -l

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p429)

    $ rbenv install 1.9.3-p429

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new version of Ruby
or install a new gem.

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p429

Verify the ruby install.  If everything was installed correctly, the `ruby -v`
command should report that version 1.9.3p429 is installed.

    $ ruby -v
    ruby 1.9.3p429 (2013-05-15 revision 40747) [x86_64-darwin12.4.0]

Install Bundler - Mac OS X
--------------------------
You'll need to use [Bundler](http://gembundler.com/) to manage gems.  Installing
a gem is also a good way to ensure that you've installed most of the Ruby
prerequisites.

First, make sue you update to the latest version of Rubygems:

    $ gem update --system

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    $ gem install bundler
    Successfully installed bundler-1.3.5
    Parsing documentation for bundler-1.3.5
    1 gem installed
    $ rbenv rehash
 
Install Ruby 2.0.0 - Mac OS X
-------------------------------------
As of this writing, Ruby 2.0.0-p195 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  Currently
there is an [issue installing Ruby 2.0.0](https://github.com/sstephenson/ruby-build/issues/305)
with the OpenSSL that ruby-build downloads, so we have to set 
`RUBY_CONFIGURE_OPTS` when calling `rbenv install` as a workaround:

    $ brew install openssl
    $ CONFIGURE_OPTS="--with-out-ext=tk,tk/* --with-openssl-dir=`brew --prefix openssl`" RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl`" rbenv install 2.0.0-p195

To verify the install:

    $ rbenv local 2.0.0-p195
    $ ruby -v
    ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-darwin12.4.0]

If you want to make Ruby 2.0.0 the global default version of ruby:

    $ rbenv global 2.0.0-p195

Upgrade Rbenv - Mac OS X
------------------------
To upgrade rbenv via homebrew:

    $ brew update
    $ brew upgrade rbenv
    $ brew upgrade ruby-build

Remove Rbenv - Mac OS X
-----------------------
Uninstall the packages you installed via homebrew:

    brew uninstall rbenv
    brew uninstall ruby-build

Remove the following directory:

    rm -rf $HOME/.rbenv

And remember to remove whatever you added to `$HOME/.bash_profile`

Linux
=====

Install Rbenv and Ruby 1.9.x - Linux
------------------------------------
Make sure the prerequisite packages are installed.

Ubuntu prerequisites:

    $ sudo apt-get update
    $ sudo apt-get install -y build-essential git
    $ sudo apt-get install -y libxml2-dev libxslt-dev libssl-dev

RHEL/CentOS prerequisites:

    $ sudo yum update
    $ sudo yum install -y git
    $ sudo yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel
    $ sudo yum install -y libyaml-devel libffi-devel openssl-devel make bzip2
    $ sudo yum install -y autoconf automake libtool bison
    $ sudo yum install -y libxml2-devel libxslt-devel 

Download the rbenv source distribution to
$HOME/.rbenv:

    $ git clone git://github.com/sstephenson/rbenv.git $HOME/.rbenv

Add $HOME/.rbenv/bin to your $PATH

    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> $HOME/.bashrc

Add `rbenv init` to your shell to enable shims and autocompletion:

    $ echo 'eval "$(rbenv init -)"' >> $HOME/.bashrc

Restart shell as a login shell so that the PATH changes take effect:

    $ exec $SHELL -l

Install the `ruby-build` plugin, which provides an
`rbenv install` command to simplify the process of compiling
and install new Ruby versions:

    $ git clone git://github.com/sstephenson/ruby-build.git $HOME/.rbenv/plugins/ruby-build

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p429)

    $ rbenv install 1.9.3-p429

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new
version of Ruby or install a new gem.

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p429

Verify the ruby install.  If everything was installed correctly, the
`ruby -v` command should report that version 1.9.3p429 is installed.

    $ ruby -v
    ruby 1.9.3p429 (2013-05-15 revision 40747) [x86_64-linux]

Install Bundler - Linux
-----------------------

You'll need to use [Bundler](http://gembundler.com/) to manage gems.
Installing a gem is also a good way to ensure that you've installed most
of the Ruby prerequisites.

First, make sure you update to the latest version of Rubygems:

    $ gem update --system

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    $ gem install bundler
    Successfully installed bundler-1.3.5
    Parsing documentation for bundler-1.3.5
    1 gem installed
    $ rbenv rehash
    
Install Ruby 2.0.0 - Linux
--------------------------
As of this writing, Ruby 2.0.0-p195 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  To install:

    $ rbenv install 2.0.0-p195

To verify the install:

    $ rbenv local 2.0.0-p195
    $ ruby -v
    ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-linux] 

If you want to make Ruby 2.0.0 the global default version of ruby:

    $ rbenv global 2.0.0-p195

Upgrade Rbenv - Linux
---------------------
Since Rbenv is a Git repository, upgrading is just a matter of refreshing the
source:

    $ cd $HOME/.rbenv
    $ git pull

Remove Rbenv - Linux
--------------------
To uninstall/remove Rbenv, remove the following directory:

    $ rm -rf $HOME/.rbenv

And remember to remove whatever you added to <code>$HOME/.bash_profile</code>
