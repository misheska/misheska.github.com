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
versions of Ruby easy.  Rbenv is a lightweight alternative to
[Ruby Version Manager (RVM)](http://rvm.io).  Rbenv does not include
any mechanism to install Ruby or manage gems, like with RVM.

Rbenv is supported on Linux and Mac OS X.  Rbenv doesn't work on Windows.
For Windows, [Pik](https://github.com/vertiginous/pik) is an Rbenv alternative.

NOTE: Rbenv is incompatible with RVM because RVM overrides the
<code>gem</code> command with a function.  If you want to switch to Rbenv,
make sure you uninstall RVM first.  First run the following commands to
uninstall RVM:
```
$ rvm implode
$ gem uninstall rvm
```
Next remove/comment out the RVM loader line in <code>.bash_profile</code>
and/or <code>.bashrc</code>

How to Install Rbenv on Mac OS X
================================
First you'll need to install a C compiler to build Ruby from source.  Just
download and install the latest version of Xcode from the App Store, if you
don't have it installed already.  Also make sure you install the *Command Line
Tools* by choosing the menu item <code>Xcode -> Preferences...</code> and click
on the *Downloads* tab.  Click on the *Install* button to download the
Command Line Tools.
{% img center /images/xcodecommandline.png 'Xcode Command Line Tools' %}

Next, you'll need to install the Homebrew package manager to all the
dependencies needed to compile Ruby from source.  While you could do this
all by hand, using a package manager is a better idea, as package
managers know how to uninstall what they install.

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

Next, install the additional dependencies to compile Ruby from source:

```
# For update-system
brew update
brew tap homebrew/dupes
# For rvm
brew install apple-gcc42
```

After apple-gcc42 is installed, download the rbenv source distribution to
$HOME/.rbenv:
```
$ git clone git://github.com/sstephenson/rbenv.git $HOME/.rbenv
```

Next, add $HOME/.rbenv/bin to your $PATH
```
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> $HOME/.bash_profile
```

Add <code>rbenv init</code> to your shell to enable shims and autocompletion:
```
$ echo 'eval "$(rbenv init -)"' >> $HOME/.bash_profile
```

Restart shell as a login shell so that the PATH changes take effect:
```
$exec $SHELL -l
```

Install the <code>ruby-build</code> plugin, which provides an
<code>rbenv install</code> command to simplify the process of compiling
and install new Ruby versions:
```
$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

Install the latest version of ruby (at the time of this writing 1.9.3-p385)
```
$ rbenv install 1.9.3-p385
```

Rebuild the shim executable
```
$ rbenv rehash
```
You'll need to run <code>rbenv rehash</code> every time you install a new
version of Ruby or install a new gem.

Set the latest version of ruby to be the default version of ruby
```
$ rbenv global 1.9.3-p385
```

Verify the ruby install
```
$ ruby -v
```

How to Upgrade Rbenv
====================
Since Rbenv is a Git repository, upgrading is just a matter of refreshing the
source:
```
$ cd $HOME/.rbenv
$ git pull
```

How to Remove Rbenv on Mac OS X
================================
To uninstall/remove Rbenv, remove the following directory:
```
rm -rf $HOME/.rbenv
```

And remember to remove whatever you added to <code>$HOME/.bash_profile</code>
