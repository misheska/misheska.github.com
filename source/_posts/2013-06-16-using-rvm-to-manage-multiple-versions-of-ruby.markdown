---
layout: post
title: "Using RVM to Manage Multiple Versions of Ruby"
date: 2013-06-16 00:35
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

[Ruby Version Manager (RVM)](http://rvm.io)  makes installing multiple
versions of Ruby easy.  It's a great way to use Ruby 1.9.x for your current
development while also being able to play with Ruby 2.0 for newer projects.
RVM also supports partitioning of Ruby gem installs via a feature called
gemsets in order to avoid versioning conflicts.  However, with the advent of
[Bundler](http://gembundler.com) most developers tend to prefer using Bundler
as the preferred way for managing gems at the application level.

RVM is supported on Linux and Mac OS X.  RVM doesn't work on Windows.
For Windows, [Pik](https://github.com/vertiginous/pik) is an RVM alternative.

Mac OS X
========

Install RVM and Ruby 1.9.x - Mac OS X
-------------------------------------
First, you'll need to install a C compiler to build Ruby from source.  Just
download and install the latest version of Xcode from the App Store, if you
don't have it already.  Also make sure you install the *Command Line Tools*
by choosing the menu item <code>Xcode -> Preferences...</code> and click
on the *Downloads* tab.  Click on the *Install* button to download the
Command Line Tools.
![Xcode Command Line Tools](/images/xcodecommandline.png)


Next, you'll need to install the Homebrew package manager.  RVM will
automatically download all dependencies if a package manager is installed.
Also, the dependencies for installing Ruby are complicated enough that it
is helpful to use a package manager to install them, so you can uninstall
them easily if necessary.  Run the following command to install Homebrew:

    ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"

Run `brew doctor` and address any issues it discovers.  When
all is well, you should see:

    $ brew doctor
    Your system is raring to brew.

After Xcode and Homebrew are installed, you can install RVM with the following
commands:

    $ \curl -L https://get.rvm.io | bash -s stable
    $ source $HOME/.rvm/scripts/rvm

Run `rvm requirements` to install additional dependencies:

    $ rvm requirements
    Installing requirements for osx, might require sudo password.
    Already up-to-date.
    Cloning into '/usr/local/Library/Taps/homebrew-dupes'...
    remote: Counting objects: 906, done.
    remote: Compressing objects: 100% (495/495), done.
    remote: Total 906 (delta 498), reused 803 (delta 411)
    Receiving objects: 100% (906/906), 157.67 KiB | 112 KiB/s, done.
    Resolving deltas: 100% (498/498), done.
    Tapped 41 formula
    Installing required packages: autoconf, automake, libtool, pkg-config, apple-gcc42, libyaml, readline, libxml2, libxslt, libksba, openssl...........................................................................................................................................................
    Updating certificates in '/usr/local/etc/openssl/cert.pem'.

Tell rvm to automatically install dependencies via Homebrew with the following command:

    $ rvm autolibs osx_brew

Next, build and install the latest version of Ruby by running the following
(this will take a long time):

    $ rvm install 1.9.3

You may get an error message saying "There was an error while trying to
resolve rubygems version for 'latest'.  Halting the installation." Just
run the install again like so to fix the issue:

    $ rvm reinstall 1.9.3

Verify the RVM install by running the following commands:

    $ rvm -h
    $ rvm list
    $ rvm use 1.9.3
    $ rvm rubygems latest

To ensure that the newer Ruby 1.9.3 is used by default instead of the
system 1.8.7 version, run the following command:

    $ rvm use 1.9.3 --default

If you'd like to manage RVM with a GUI, check out [JewelryBox](http://jewelrybox.unfiniti.com/):

![Jewelry Box](/images/jewelrybox.png)

Install Ruby 1.8.7 - Mac OS X
-----------------------------
For legacy GUI support, Ruby 1.8.7 has some dependencies on tcl/tk, which
Mountain Lion no longer installs by default (now that X11 is an optional
install).  To compile Ruby 1.8.7 without tcl/tk support, use the following
command overrides on <code>rvm install</code>:

    $ rvm install 1.8.7 --with-gcc=clang --without-tcl --without-tk

To compile Ruby 1.8.7 with tcl/tk support, install X11 via
[http://xquartz.macosforge.org/landing](http://xquartz.macosforge.org/landing)
then compile Ruby 1.8.7 with the following:

    $ export CPPFLAGS=-I/opt/X11/include
    $ CC=/usr/local/bin/gcc-4.2 rvm install 1.8.7

Install Ruby 2.0.0 - Mac OS X
-----------------------------
Installing Ruby 2.0 is very straightforward, just run the following:

    $ rvm install 2.0.0

Verify the Ruby 2.0 install by running the following commands:

    $ rvm use 2.0.0
    $ ruby -v
    ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-darwin12.3.0]

If you want to make Ruby 2.0 your default version of ruby, run the following:

    $ rvm use 2.0.0 --default

Remove RVM - Mac OS X
---------------------
Should you want to uninstall/remove RVM, it's pretty easy.  First, run
the following commands:

    $ rvm implode
    $ gem uninstall rvm

Then just follow the instructions from `rvm implode`:

* Delete the following files, if they exist:
`/etc/rvmrc`
`$HOME/.rvmrc`

* Also, remove the lines sourcing RVM scripts from 
`$HOME/.bash_profile` `/etc/zprofile` and 
`/etc/profile.d/rvm.sh` if they exist.  (A reboot doesn't hurt
after you do this, just to make sure).

To uninstall Homebrew, run the following:

    cd `brew --prefix`
    brew install libtool
    rm -rf Cellar
    rm `git ls-files`
    rm -r Library/Homebrew Library/Aliases Library/Formula Library/Contributions
    rm -rf .git
    rm -rf ~/Library/Caches/Homebrew

Linux
=====

Install RVM and Ruby 1.9.x - Linux
----------------------------------
RVM installation is more straightforward on Linux, as the Ruby source was
designed for the GCC compiler that ships with any Linux distribution.

First install <code>curl</code>, so that you can fetch the RVM script.
For Ubuntu, run
the following command:

    $ sudo apt-get update
    $ sudo apt-get install -y curl

For RedHat/CentOS:

    $ sudo yum update
    $ sudo yum install -y curl


With curl installed, run the RVM install with the following commands:

    $ \curl -L https://get.rvm.io | bash -s stable
    $ source $HOME/.rvm/scripts/rvm

On Ubuntu, run `rvm requirements` to install additional dependencies:

    $ rvm requirements

For CentOS, `rvm requirements` will need to download packages from EPEL, so
add the `--verify-downloads 1` parameter after the `rvm requirements` command:

    $ rvm requirements --verify-downloads 1

Next build and install Ruby 1.9.3 (this will take awhile):

    $ rvm install 1.9.3
    $ rvm use 1.9.3
    $ rvm rubygems latest

Verify rvm install:

    $ rvm -h
    $ rvm list
    $ rvm use 1.9.3
    $ ruby -v
    ruby 1.9.3p429 (2013-05-15 revision 40747) [x86_64-linux]

Install Ruby 2.0.0 - Linux
--------------------------
Installing Ruby 2.0 is very straightforward, just run the following:

    $ rvm install 2.0.0

Verify the Ruby 2.0 install by running the following commands:

    $ rvm use 2.0.0
    $ ruby -v
    ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-linux]

If you want to make Ruby 2.0 your default version of ruby, run the following:

    $ rvm use 2.0.0 --default

Remove RVM - Linux
------------------
Should you want to uninstall/remove RVM, it's pretty easy.  First, run
the following commands:

    $ rvm implode
    $ gem uninstall rvm

Then just follow the instructions from <code>rvm implode</code>:

* Delete the following files, if they exist:
`/etc/rvmrc`
`$HOME/.rvmrc`

* Also, remove the lines sourcing RVM scripts from
`$HOME/.bash_profile` `/etc/zprofile` and
`/etc/profile.d/rvm.sh` if they exist.  (A reboot doesn't hurt
after you do this, just to make sure).
