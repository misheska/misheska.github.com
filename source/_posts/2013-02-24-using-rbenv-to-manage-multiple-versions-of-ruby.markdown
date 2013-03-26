---
layout: post
title: "Using Rbenv to manage multiple versions of Ruby"
date: 2013-02-24 22:47
comments: true
categories: ruby
---
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

How to Install Rbenv on Mac OS X
================================
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

Install the latest version of ruby (at the time of this writing 1.9.3-p392)

    $ rbenv install 1.9.3-p392

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p392

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new
version of Ruby or install a new gem.

Verify the ruby install

    $ ruby -v
    ruby 1.9.3p392 (2013-02-22 revision 39386) [x86_64-darwin12.3.0]
    
How to install Ruby 2.0.0 on Mac OS X
=====================================
As of this writing, Ruby 2.0.0-p0 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  To install:

    $ rbenv install 2.0.0-p0

To verify the install:

    $ rbenv local 2.0.0-p0
    $ ruby -v

If you want to make Ruby 2.0.0 the global default version of ruby:

    $ rbenv global 2.0.0-p0

How to Upgrade Rbenv on Mac OS X
================================
To upgrade rbenv via homebrew:

    $ brew update
    $ brew upgrade rbenv
    $ brew upgrade ruby-build

How to Remove Rbenv on Mac OS X
================================
Uninstall the packages you installed via homebrew:

    brew uninstall rbenv
    brew uninstall ruby-build

And remove the following directory:

    rm -rf $HOME/.rbenv

And remember to remove whatever you added to `$HOME/.bash_profile`

How to Install RVM on Linux
===========================
Make sure the prerequisite packages are installed.  For Ubuntu:

    $ sudo apt-get install build-essential git

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

Install the latest version of ruby (at the time of this writing 1.9.3-p392)

    $ rbenv install 1.9.3-p392

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new
version of Ruby or install a new gem.

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p392

Verify the ruby install

    $ ruby -v

    
How to install Ruby 2.0.0 on Linux
==================================
As of this writing, Ruby 2.0.0-p0 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  To install:

    $ rbenv install 2.0.0-p0

To verify the install:

    $ rbenv local 2.0.0-p0
    $ ruby -v

If you want to make Ruby 2.0.0 the global default version of ruby:

    $ rbenv global 2.0.0-p0

How to Upgrade Rbenv on Linux
================================
Since Rbenv is a Git repository, upgrading is just a matter of refreshing the
source:

    $ cd $HOME/.rbenv
    $ git pull

How to Remove Rbenv on Linux
================================
To uninstall/remove Rbenv, remove the following directory:

    $ rm -rf $HOME/.rbenv

And remember to remove whatever you added to <code>$HOME/.bash_profile</code>
