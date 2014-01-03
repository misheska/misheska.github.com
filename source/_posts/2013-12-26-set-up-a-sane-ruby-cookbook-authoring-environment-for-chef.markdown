---
layout: post
title: "Set up a Sane Ruby Cookbook Authoring Environment for Chef on Mac OS X, Linux and Windows"
date: 2013-12-26 23:32
comments: true
categories: [Ruby, Chef]
---
* list element with functor item
{:toc}

_Updated January 2, 2014_

* _Per Seth Vargo switched from rbenv to chruby_
* _Switched from SublimeText 2 to SublimeText 3 - it works with SublimeChef_

You will need to set up a sane Ruby 1.9.x development to support your
Chef cookbook authoring activity on Mac OS X, Linux or Windows.  In this
Ruby environment, you will manage all the required Ruby Gem libraries
necessary to support Chef cookbook authoring.  The
[LearnChef](https://learnchef.opscode.com/quickstart/workstation-setup/)
site recommends that you piggyback the Chef client's omnibus Ruby
environment to start learning how to write Chef cookbooks.  This
article assumes that you want to go beyond the basics, where you'll need
a Ruby environment dedicated for Chef cookbook development.

There are many different ways to set up a sane Ruby environment.  This
article covers how to set up a sane Ruby environment for Chef using
[Chruby](https://github.com/postmodern/chruby) for Mac OS X/Linux and
[RubyInstaller](http://rubyinstaller.org) for Windows.  The setup
covered in this article should work for most people wanting to write
Chef cookbooks.  If you are more experienced with Ruby development, you
may want to roll your own Ruby environment in another fashion.  The only
requirement is that Ruby 1.9.x must be used, the Chef client currently
does not support Ruby 2.x.

Mac OS X
========

Out of the box, Ruby does not provide a mechanism to support multiple
installed versions.  [Chruby](https://github.com/postmodern/chruby)
makes it easy to manage multiple versions of Ruby.  It's a great way to set up
a dedicated Ruby 1.9.x environment with all the required Gem libraries for
Chef cookbook development.

- - -

__NOTE:__ Before trying to install
[Chruby](https://github.com/postmodern/chruby)
verify that you do not have another popular Ruby virtualization manager
installed - [RVM](http://rvm.io).  If you try to run the following `rvm`
command, it should say `command not found`:

    $ rvm --version
    -bash: rvm: command not found

If you want to switch to Chruby (which is recommended), make sure that you
[completely remove RVM first](http://stackoverflow.com/questions/3558656/how-can-i-remove-rvm-ruby-version-manager-from-my-system)
(as Chruby and RVM cannot coexist because RVM overrides the `gem` command with
a function specific to RVM).

- - -

Install the Xcode Command Line Tools - Mac OS X
-----------------------------------------------

First you'll need to install a C compiler and the Xcode Command Line tools
to build Ruby from source.  If you are using the latest version of Mac OS X
Mavericks 10.9, it has support for downloading the Xcode command line tools
directly from a Terminal window.  Run the following on a command line:

    $ xcode-select --install

You will be prompted to either click on `Install` to just install the command
line developer tools or click on `Get Xcode` to install both Xcode and the
command line developer tools.  It can be handy to have Xcode as well, but
it is a rather large multi-gigabyte download and not really necessary for
Ruby development.  So if you want to get going quickly, just click on the
`Install` button:

{% img center /images/xcodeselect.png xcode-select %}

Install the Homebrew Package Manager - Mac OS X
-----------------------------------------------

Next, you'll need to install the [Homebrew package manager](http://brew.sh/)
to get all the dependencies needed to compile Ruby from source.  While you
could manage these dependencies by hand, using a package manager is a better
idea, as package managers know how to uninstall what they install.

First verify that you __DO NOT__ currently have homebrew installed.
`brew --version` should report `command not found`.

    $ brew --version
    -bash: brew: command not found

If you already have Homebrew installed, just
[Update Homebrew and Rbenv](http://misheska.com/blog/2013/06/15/using-rbenv-to-manage-multiple-versions-of-ruby/#upgrade-rbenv---mac-os-x)
and skip to the next section.

- - -

__NOTE:__ Before trying to install [Homebrew](http://brew.sh) verify that
you do not have another popular package manager installed - MacPorts.
If you try to run the following `port` command, it should say
`command not found`:

    $ port --version
    -bash: port: command not found

If MacPorts is already installed, make sure that you
[completely remove MacPorts from your system](http://guide.macports.org/chunked/installing.macports.uninstalling.html)
before trying to install Homebrew.

- - -

Run the following command to install Homebrew:

    $ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

Run `brew doctor` and address any issues it discovers.  When
all is well, you should see:

    $ brew doctor
    Your system is raring to brew.

Install Apple GCC 4.2 - Mac OS X
--------------------------------

Next, install the additional dependencies to compile Ruby from source:

    # For update-system
    brew update
    # Add the system duplicate formulae
    brew tap homebrew/dupes
    # Install legacy C compiler for building Ruby
    brew install apple-gcc42

Install Chruby and Ruby-Build via Homebrew - Mac OS X
------------------------------------------------------

Now install `chruby` and `ruby-install` via Homebrew:

    $ brew update
    $ brew install chruby
    $ brew install ruby-build

Add `chruby.sh` to your shell to enable chruby:

    $ echo 'source /usr/local/share/chruby/chruby.sh' >> $HOME/.bash_profile

Add `auto.sh` to your shell to enble auto-switching of Rubies specified by
`.ruby-version files:

    $ echo 'source /usr/local/share/chruby/auto.sh' >> $HOME/.bash_profile

Compile Ruby 1.9.x from source - Mac OS X
-----------------------------------------

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p484)

    $ ruby-build 1.9.3-p484 --install-dir ~/.rubies/ruby-1.9.3-p484

Reload `.bashrc` with these new settings:

    $ source $HOME/.bash_profile    

To switch to the Ruby required for Chef:

    $ chruby ruby-1.9

To make this version the default version of Ruby, simply add this command
to your $HOME/.bash_profile

    $ echo 'chruby ruby-1.9' >> $HOME/.bash_profile

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
    Fetching: rubygems-update-2.2.0.gem (100%)
    Successfully installed rubygems-update-2.2.0
    Installing RubyGems 2.2.0
    RubyGems 2.2.0 installed
    ...

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    $ gem install bundler
    Successfully installed bundler-1.5.1
    Parsing documentation for bundler-1.5.1
    1 gem installed

Install Sublime Text 3 (Optional) - Mac OS X
--------------------------------------------

Miguel Cabeça has written an excellent plugin for the Sublime Text editor
called [SublimeChef](https://github.com/cabeca/SublimeChef), which is the
closest thing to an Integrated Development Environment (IDE) that exists
for Chef.  Sublime Text is costs $70 for a license, but has no restriction
on the length of a trial period, so feel free try out Sublime Text to see
if it works for you.

Download and install SublimeText 3 for your platform via
<http://www.sublimetext.com/3>

### Install Sublime Text 3 Package Control (Optional) - Mac OS X

In order to install the SublimeChef plugin, first you need to install
Sublime Text Package Control.

* Download [Package Control.sublime-package](https://sublime.wbond.net/Package%20Control.sublime-package)

* Choose `Preferences > Browse Packages...` from the Sublime Text 3 menu

* Browse up a folder, then navigate into `Installed Packages/`

* Copy `Package Control.sublime-package` file you downloaded into this
`Sublime Text 3\Installed Packages` directory

* Restart Sublime Text 3 to enable Package Control

Once Package Control is installed successfully, you should be able to
display the Command Palette by pressing `COMMAND+SHIFT+P`:

{% img center /images/sublime3commandpalette-mac.png [Sublime Text 3 Command Palette] %}

### Install Sublime Chef (Optional) - Mac OS X

After pressing `CTRL+SHIFT+P` to display the Command Palette, start typing
`install` to select `Package Control: Install Package`:

{% img center /images/sublime3installpackagecontrol-mac.png [Sublime Text 3 Package Control] %}

Then type `chef` to display the SublimeChef entry - click to install:

{% img center /images/sublime3sublimechef-mac.png [SublimeChef install] %}

Miguel created the following
[demo video](http://www.youtube.com/watch?v=4VtDj_ar1Xg)
to show how SublimeChef can be used.

Linux
=====

Install Prerequisite Packages - Linux
-------------------------------------
Make sure the prerequisite packages are installed.

### Ubuntu prerequisites:

    $ sudo apt-get update
    $ sudo apt-get install -y build-essential git
    $ sudo apt-get install -y libxml2-dev libxslt-dev libssl-dev

### RHEL/CentOS prerequisites:

    $ sudo yum update
    $ sudo yum install -y git
    $ sudo yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel
    $ sudo yum install -y libyaml-devel libffi-devel openssl-devel make bzip2
    $ sudo yum install -y autoconf automake libtool bison
    $ sudo yum install -y libxml2-devel libxslt-devel

Install Chruby and Ruby-Install - Linux
---------------------------------------

Download the chruby source distribution:

    $ wget -O chruby-0.3.8.tar.gz https://github.com/postmodern/chruby/archive/v0.3.8.tar.gz

Extract chruby and install:

    $ tar xzvf chruby-0.3.8.tar.gz
    $ cd chruby-0.3.8/
    $ sudo make install

Feel free to remove the chruby source dir after installing:

    $ cd ..
    $ rm chruby-0.3.8.tar.gz
    $ rm -rf chruby-0.3.8

Install ruby-build:

    $ git clone https://github.com/sstephenson/ruby-build.git
    $ cd ruby-build
    $ sudo ./install.sh

Feel free to remove the ruby-build source dir after installing:

    $ cd ..
    $ rm -rf ruby-build

Add `chruby.sh` to your shell to enable chruby:

    $ echo 'source /usr/local/share/chruby/chruby.sh' >> $HOME/.bashrc

Add `auto.sh` to your shell to enble auto-switching of Rubies specified by
`.ruby-version files:

    $ echo 'source /usr/local/share/chruby/auto.sh' >> $HOME/.bashrc

Compile Ruby 1.9.x from source - Linux
-----------------------------------------

Install the latest version of ruby 1.9.x (at the time of this writing 1.9.3-p484)

    $ ruby-build 1.9.3-p484 --install-dir ~/.rubies/ruby-1.9.3-p484

Reload `.bashrc` with these new settings:

    $ source $HOME/.bashrc

To switch to the Ruby required for Chef:

    $ chruby ruby-1.9

To make this version the default version of Ruby, simply add this command
to your $HOME/.bashrc

    $ echo 'chruby ruby-1.9' >> $HOME/.bashrc

Verify the ruby install.  If everything was installed correctly, the `ruby -v`
command should report that version 1.9.3p484 is installed.

    $ ruby -v
    ruby 1.9.3p484 (2013-11-22 revision 43786) [x86_64-darwin13.0.0]

Install Bundler - Linux
-----------------------

You'll need to use [Bundler](http://gembundler.com/) to manage gems.
Installing a gem is also a good way to ensure that you've installed most
of the Ruby prerequisites.

First, make sure you update to the latest version of Rubygems:

    $ gem update --system
    Updating rubygems-update
    Fetching: rubygems-update-2.2.0.gem (100%)
    Successfully installed rubygems-update-2.2.0
    Installing RubyGems 2.2.0
    RubyGems 2.2.0 installed
    ...

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    $ gem install bundler
    Successfully installed bundler-1.5.0
    Parsing documentation for bundler-1.5.0
    1 gem installed

Install Sublime Text 3 (Optional) - Linux
-----------------------------------------

Miguel Cabeça has written an excellent plugin for the Sublime Text editor
called [SublimeChef](https://github.com/cabeca/SublimeChef), which is the
closest thing to an Integrated Development Environment (IDE) that exists
for Chef.  Sublime Text is costs $70 for a license, but has no restriction
on the length of a trial period, so feel free try out Sublime Text to see
if it works for you.

Download and install SublimeText 3 for your platform via
<http://www.sublimetext.com/3>

### Install Sublime Text 3 Package Control (Optional) - Linux

In order to install the SublimeChef plugin, first you need to install
Sublime Text Package Control.

* Download [Package Control.sublime-package](https://sublime.wbond.net/Package%20Control.sublime-package)

* Choose `Preferences > Browse Packages...` from the Sublime Text 3 menu

* Browse up a folder, then navigate into `Installed Packages/`

* Copy `Package Control.sublime-package` file you downloaded into this
`Sublime Text 3/Installed Packages` directory

* Restart Sublime Text 3 to enable Package Control

Once Package Control is installed successfully, you should be able to
display the Command Pallete by pressing `CTRL+SHIFT+P`:

{% img center /images/sublime3commandpalette-ubuntu.png [Sublime Text 3 Command Palette] %}

### Install Sublime Chef (Optional) - Linux

After pressing `CTRL+SHIFT+P` to display the Command Pallette, start typing
`install` to select `Package Control: Install Package`:

{% img center /images/sublime3installpackagecontrol.png [Sublime Text 3 Package Control] %}

Then type `chef` to display the SublimeChef entry - click to install:

{% img center /images/sublime3sublimechef-ubuntu.png [SublimeChef install] %}

Miguel created the following
[demo video](http://www.youtube.com/watch?v=4VtDj_ar1Xg)
to show how SublimeChef can be used.

Windows
=======

There is no need to install a Ruby version manager on Windows, like there
is for Mac OS X or Linux.  In fact, the chruby version manager does not
work on Windows.  Instead, you'll use the 
[RubyInstaller for Windows](http://rubyinstaller.org/) which
can install different versions of Ruby on the same machine.

Install Ruby 1.9.x - Windows
----------------------------

Download and install the latest Windows RubyInstaller for Ruby 1.9.x from
<http://rubyinstaller.org/downloads>
(version 1.9.3-p484 as of this writing):

{% img center /images/rubyinstaller.png [RubyInstaller] %}

Verify that Ruby was installed correctly by running the following from a
Command Prompt:

    > ruby -v
    ruby 1.9.3p484 (2013-11-22) [i386-mingw32]

Install Ruby DevKit - Windows
-----------------------------

Download and install the Ruby Development Kit for use with Ruby 1.8.7 and
1.9.3.

{% img center /images/devkit.png [Ruby DevKit] %}

Extract the archive to `C:\Ruby\DevKit`:

{% img center /images/devkitextract.png [Extract DevKit to C:\Ruby\Devkit]  %}

Enhance Rubies to use the DevKit - Windows
------------------------------------------

Run `dk.rb init` to generate a `config.yml` which includes all the installed
Rubies to be enhanced to use the DevKit:

    > cd /d c:\Ruby\DevKit
    > ruby dk.rb init

If you want to review the list of Rubies before installing, run
`dk.rb review`:

    > cd /d c:\Ruby\DevKit
    > ruby dk.rb review

Then run `dk.rb` to DevKit enhance your installed Rubies:

    > cd /d c:\Ruby\DevKit
    > ruby dk.rb install

Finally run `devkitvars` to add the DevKit tools to your command shell's
PATH and try to get the version of `gcc` to verify that the tools
installed properly:

    > c:\Ruby\DevKit\devkitvars.bat
    Adding the DevKit to PATH...
    > gcc --version
    gcc (tdm-1) 4.5.2
    Copyright (C) 2010 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.  There is NO
    warranty; not even for MERCHANTABILITY or FITNESS OR A PARTICULAR PURPOSE.

Install Bundler - Windows
-------------------------

You'll need to use [Bundler](http://gembundler.com/) to manage gems.  Installing
a gem is also a good way to ensure that you've installed most of the Ruby
prerequisites.

First, make sure you update to the latest version of Rubygems:

    > c:\Ruby\DevKit\devkitvars.bat
    Adding the DevKit to Path...
    > gem update --system
    Fetching: rubygems-update-2.2.0.gem (100%)
    Successfully installed ruygems-update-2.2.0
    Installing RubyGems 2.2.0
    RubyGems 2.2.0 installed
    ...

Then install the `bundler` gem.  If the `gem install` command reports
`Successfully installed` you're good to go:

    > gem install bundler
    Successfully installed bundler-1.5.0
    Parsing documentation for bundler-1.5.0
    1 gem installed

Install Git - Windows
---------------------

While using source control is a recommended best practice, Chef does not
require Git.  If you use some other source control system besides Git,
feel free to install it instead of Git.  Use of Git source control just
happens to be more common than others with Chef.

Download the latest stable release of the Git for Windows install from
<http://git-scm.com/downloads>

{% img center /images/wingit.png [Download for Windows] %}

Run the Git Windows install accepting the default choices, except on the
PATH setup dialog.  You'll need to include the Unix tools on your Windows
command prompt for some Chef commands to work:

{% img center /images/wingitunixtools.png [Run Git and Included Unix tools from the Windows Command Prompt] %}

Verify that Git installed correcting by running the following on a command
prompt:

    > git --version
    git version 1.8.4.msysgit.0

Install Sublime Text 3 (Optional) - Windows
-------------------------------------------

Miguel Cabeça has written an excellent plugin for the Sublime Text editor
called [SublimeChef](https://github.com/cabeca/SublimeChef), which is the
closest thing to an Integrated Development Environment (IDE) that exists
for Chef.  Sublime Text is costs $70 for a license, but has no restriction
on the length of a trial period, so feel free try out Sublime Text to see
if it works for you.

Download and install SublimeText 3 for your platform via
<http://www.sublimetext.com/3>

### Install Sublime Text 3 Package Control (Optional) - Windows

In order to install the SublimeChef plugin, first you need to install
Sublime Text Package Control.

* Download [Package Control.sublime-package](https://sublime.wbond.net/Package%20Control.sublime-package)

* Choose `Preferences > Browse Packages...` from the Sublime Text 3 menu

* Browse up a folder, then navigate into `Installed Packages/`

* Copy `Package Control.sublime-package` file you downloaded into this
`Sublime Text 3\Installed Packages` directory

* Restart Sublime Text 3 to enable Package Control

Once Package Control is installed successfully, you should be able to
display the Command Pallete by pressing `CTRL+SHIFT+P`:

{% img center /images/sublime2commandpalette.png [Sublime Text 2 Command Palette] %}

### Install Sublime Chef (Optional) - Windows

After pressing `CTRL+SHIFT+P` to display the Command Pallette, start typing
`install` to select `Package Control: Install Package`:

{% img center /images/sublime2installpackagecontrol.png [Sublime Text 3 Package Control] %}

Then type `chef` to display the SublimeChef entry - click to install:

{% img center /images/sublime2sublimechef.png [SublimeChef install] %}

Miguel created the following
[demo video](http://www.youtube.com/watch?v=4VtDj_ar1Xg)
to show how SublimeChef can be used.
