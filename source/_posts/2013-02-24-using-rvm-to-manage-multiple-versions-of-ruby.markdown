---
layout: post
title: "Using RVM to manage multiple versions of Ruby"
date: 2013-02-24 19:34
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

[Ruby Version Manager (RVM)](http://rvm.io)  makes installing multiple
versions of Ruby easy.  RVM also supports partitioning of Ruby gem installs
via a feature called gemsets in order to avoid versioning conflicts.  However,
with the advent of [Bundler](http://gembundler.com) most developers tend to
prefer using Bundler as the preferred way for managing gems at the
application level, so this feature of RVM isn't of as much use as it was
in the past.

RVM is supported on Linux and Mac OS X.  RVM doesn't work on Windows.
For Windows, [Pik](https://github.com/vertiginous/pik) is an RVM alternative.

How to Install RVM on Mac OS X
==============================
First, you'll need to install a C compiler to build Ruby from source.  Just
download and install the latest version of Xcode from the App Store, if you
don't have it already.  Also make sure you install the *Command Line Tools*
by choosing the menu item <code>Xcode -> Preferences...</code> and click
on the *Downloads* tab.  Click on the *Install* button to download the
Command Line Tools.
{% img center /images/xcodecommandline.png 'Xcode Command Line Tools' %}

Next, you'll need to install the Homebrew package manager.  RVM will
automatically download all dependencies if a package manager is installed.
Run the following command to install Homebrew:

```
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```

Next run <code>brew doctor</code> and address any issues it discovers.  When
all is well, you should see:

```
$ brew doctor
Your system is raring to brew.
```

After Xcode and Homebrew are installed, you can install RVM with the following
commands:

```
$ \curl -#L https://get.rvm.io | bash -s stable
$ source $HOME/.rvm/scripts/rvm
```

Run <code>rvm requirements</code> and do what it says to install the additional
dependencies.  On my system, I ran the following:

```
# For update-system
brew update
brew tap homebrew/dupes
# For rvm
brew install apple-gcc42
```

Next, build and install the latest version of Ruby by running the following
(this will take a long time):
```
$ rvm install 1.9.3
$ rvm use 1.9.3
$ rvm rubygems latest
```

Verify the RVM install by running the following commands:
```
$ rvm -h
$ rvm list
$ rvm use 1.9.3
```

To ensure that the newer Ruby 1.9.3 is used by default instead of the
system 1.8.7 version, run the following command:
```
rvm use 1.9.3 --default
```

How to Remove RVM on Mac OS X
=============================
Should you want to uninstall/remove RVM, it's pretty easy.  First, run
the following commands:

```
$ rvm implode
$ gem uninstall rvm
```

Next, just follow the instructions <code>rvm implode</code> displays.

Delete the following files, if they exist:
<code>/etc/rvmrc</code>
<code>$HOME/.rvmrc</code>

Also, remove the lines sourcing RVM scripts from 
<code>$HOME/.bash_profile</code> <code>/etc/zprofile</code> and 
<code>/etc/profile.d/rvm.sh</code> if they exist.  (A reboot doesn't hurt
after you do this, just to make sure).

To uninstall Homebrew, run the following:

```
cd `brew --prefix`
brew install libtool
rm -rf Cellar
rm `git ls-files`
rm -r Library/Homebrew Library/Aliases Library/Formula Library/Contributions
rm -rf .git
rm -rf ~/Library/Caches/Homebrew
```

How to Install RVM on Linux
===========================
RVM installation is more straightforward on Linux, as the Ruby source was
designed for the GCC compiler that ships with any Linux distribution.

First install <code>curl</code>, so that you can fetch the RVM script.
For Ubuntu, run
the following command:
```
$ sudo apt-get install curl
```
For RedHat/CentOS:
```
$ sudo yum install curl
```

With curl installed, run the RVM install with the following commands:
```
$ \curl -L https://get.rvm.io | bash -s stable
$ source $HOME/.rvm/scripts/rvm
```

Next run the <code>rvm requirements</code> command to get rvm to tell us what
other packages need to be installed for Ruby to work:
```
$ rvm requirements
...
# For Ruby / Ruby HEAD (MRI, Rubinius, & REE), install the following:
ruby: /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev 
curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev
libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
```

Run the appropriate command for your Linux distribution to install the
prerequisite packages.  For example, on Ubuntu, you would run:
```
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev \
curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 \
libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison  \
subversion pkg-config
```

Next build and install Ruby 1.9.3 (this will take awhile):
```
$ rvm install 1.9.3
$ rvm use 1.9.3
$ rvm rubygems latest
```

Verify rvm install:
```
$ rvm -h
$ rvm list
$ rvm use 1.9.3
```

How to Remove RVM on Linux
==========================
Should you want to uninstall/remove RVM, it's pretty easy.  First, run
the following commands:

```
$ rvm implode
$ gem uninstall rvm
```

Next, just follow the instructions <code>rvm implode</code> displays.

Delete the following files, if they exist:
<code>/etc/rvmrc</code>
<code>$HOME/.rvmrc</code>

Also, remove the lines sourcing RVM scripts from
<code>$HOME/.bash_profile</code> <code>/etc/zprofile</code> and
<code>/etc/profile.d/rvm.sh</code> if they exist.  (A reboot doesn't hurt
after you do this, just to make sure).
