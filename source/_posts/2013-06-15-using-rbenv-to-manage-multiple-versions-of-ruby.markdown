---
layout: post
title: "Using Rbenv to Manage Multiple Versions of Ruby"
date: 2013-06-15 15:53
comments: true
categories: ruby
---
* list element with functor item
{:toc}

_Updated December 15th, 2013:_

* _Updated install link for homebrew, it has changed_
* _Updated Xcode install instructions for Mac OS X Mavericks_
* _Bumped ruby 1.9.3 version from 1.9.3p448 to 1.9.3p484_
* _Bumped ruby 2.0.0 version from 2.0.0p247 to 2.0.0p353_

_Updated July 9th, 2013:_

* _Bumped ruby 1.9.3 version from 1.9.3p429 to 1.9.3p448_
* _Bumped ruby 2.0.0 version from 2.0.0p195 to 2.0.0p247_
* _Removed openssl workaround for compiling ruby 2.0.0 on Mac OS X - this has been fixed_

Out of the box, Ruby does not provide a mechanism to support multiple
installed versions.  [Rbenv](https://github.com/sstephenson/rbenv/) makes
managing multiple versions of Ruby easy.  It's a great way to work on
current development projects using Ruby 1.9.x and be able to switch to
Ruby 2.0.x for new work.  Rbenv is a lightweight alternative to
[Ruby Version Manager (RVM)](http://rvm.io).  Rbenv does not include
a way to manage gems, like with RVM, though most people prefer to use
[Bundler](http://bundler.io) gem management instead.

Rbenv is supported on Linux and Mac OS X.  Rbenv doesn't work on Windows (and
isn't really necessary on Windows as there is no system Ruby on Windows).
On Windows, the [RubyInstaller](http://rubyinstaller.org) and/or
[Pik](https://github.com/vertiginous/pik) are both good alternatives to
work with multiple versions of Ruby on the same box.

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
First you'll need to install a C compiler and some other command line tools
to build Ruby from source.  You'll need to install the Xcode Command Line
Tools:  

### Mac OS X Mavericks 10.9

If you are using the latest version of Mac OS X Mavericks, it has support
for downloading the Xcode command line tools directly from a Terminal window.
Run the following on a command line:

    $ xcode-select --install

You will be prompted to either click on `Install` to just install the command
line developer tools or click on `Get Xcode` to install both Xcode and the
command line developer tools.  It can be handy to have Xcode as well, but
it is a rather large multi-gigabyte download and not really necessary for
Ruby development.  So if you want to get going quickly, just click on the
`Install` button:

![xcode-select](/images/xcodeselect.png)

### Mac OS X Snow Leopard 10.8 (and earlier)

Versions of Mac OS X prior to Mavericks, like Snow Leopard do not have
built-in support for downloading the Xcode command line developer tools, but you
can download them directly from the Apple Developer web site
[developer.apple.com/downloads](http://developer.apple.com/downloads).
Here are the current direct download links for the Xcode command line tools
as of this writing.  Download the applicable package for your version of
Mac OS X:

* [Xcode Command Line Tools for Mac OS X Mountain Lion 10.8](http://devimages.apple.com/downloads/xcode/command_line_tools_os_x_mountain_lion_for_xcode_october_2013.dmg)
* [Xcode Command Line Tools for Mac OS X Lion 10.7](http://devimages.apple.com/downloads/xcode/command_line_tools_for_xcode_os_x_lion_april_2013.dmg)

Once you have downloaded the .dmg file, double-click on the .dmg to open it,
then double-click on the .mpkg file to run the Command Line Tools installer:

![xcode-select](/images/commandlinetools.png)

If you have time to install Xcode, it's handy to have it around, though not
really required for Ruby development.  If you are running Mac OS X Mountain
Lion 10.8, you will need to make sure that you have OS X version 10.8.4 or
later to install Xcode, otherwise the App Store will issue the following
complaint:

![xcode-select](/images/xcode1084.png)

Download and install the latest version of Xcode from the App Store.
Then install the *Command Line Tools* by choosing the menu item
 <code>Xcode -> Preferences...</code> to display the `Downloads` dialog.
Click on the *Downloads* tab, then click on the download arrow to the right
of the `Command Line Tools` component to install.

![Xcode Command Line Tools](/images/xcodecommandline2.png)

Next, you'll need to install the Homebrew package manager to get all the
dependencies needed to compile Ruby from source.  While you could manage
these dependencies by hand, using a package manager is a better idea, as
package managers know how to uninstall what they install.

Run the following command to install Homebrew:

    $ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

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

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p484)

    $ rbenv install 1.9.3-p484

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new version of Ruby
or install a new gem.

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p484

Verify the ruby install.  If everything was installed correctly, the `ruby -v`
command should report that version 1.9.3p484 is installed.

    $ ruby -v
    ruby 1.9.3p484 (2013-11-22 revision 43786) [x86_64-darwin13.0.0]

Install Bundler - Mac OS X
--------------------------
You'll need to use [Bundler](http://gembundler.com/) to manage gems.  Installing
a gem is also a good way to ensure that you've installed most of the Ruby
prerequisites.

First, make sure you update to the latest version of Rubygems:

    $ gem update --system
    Updating rubygems-update
    Fetching: rubygems-update-2.1.11.gem (100%)
    Successfully installed rubygems-update-2.1.11
    Installing RubyGems 2.1.11
    RubyGems 2.1.11 installed
    ...

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    $ gem install bundler
    Successfully installed bundler-1.3.5
    Parsing documentation for bundler-1.3.5
    1 gem installed
    $ rbenv rehash
 
Install Ruby 2.0.0 - Mac OS X
-------------------------------------
As of this writing, Ruby 2.0.0-p353 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  To install:

    $ rbenv install 2.0.0-p353

To verify the install:

    $ rbenv local 2.0.0-p353
    $ ruby -v
    ruby 2.0.0p353 (2013-11-22 revision 43784) [x86_64-darwin13.0.0]

If you want to make Ruby 2.0.0 the global default version of ruby:

    $ rbenv global 2.0.0-p353

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

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p484)

    $ rbenv install 1.9.3-p484

Rebuild the shim executable

    $ rbenv rehash

You'll need to run `rbenv rehash` every time you install a new
version of Ruby or install a new gem.

Set the latest version of ruby to be the default version of ruby

    $ rbenv global 1.9.3-p484

Verify the ruby install.  If everything was installed correctly, the
`ruby -v` command should report that version 1.9.3p484 is installed.

    $ ruby -v
    ruby 1.9.3p484 (2013-11-22 revision 43786) [x86_64-linux]

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
As of this writing, Ruby 2.0.0-p353 is the latest version of Ruby 2.0.0.
Use `rbenv install --list` to print out the available versions.  To install:

    $ rbenv install 2.0.0-p353

To verify the install:

    $ rbenv local 2.0.0-p353
    $ ruby -v
    ruby 2.0.0p353 (2013-11-22 revision 43784) [x86_64-linux]

If you want to make Ruby 2.0.0 the global default version of ruby (but
currently Chef requires Ruby 1.9.x):

    $ rbenv global 2.0.0-p353

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
