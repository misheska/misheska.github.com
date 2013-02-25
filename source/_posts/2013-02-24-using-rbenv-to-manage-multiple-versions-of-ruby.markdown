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
system-wide version of Ruby is over a year old (ruby 1.8.7p358).  Most
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
First you'll need to install a C compiler to build Ruby from source.
While you might think that this is just a matter of installing the latest
version of Xcode, which comes with a C compiler, unfortunately the Ruby
source code is not fully compatible with either the LLVM-GCC or Clang C
compilers shipping with Xcode 4.2 and later.  For more explanation of the
issue, refer to this article [HERE](http://woss.name/2012/01/24/how-to-install-a-working-set-of-compilers-on-mac-os-x-10-7-lion/)

Run the following command to see if you have the GCC compiler installed,
which the Ruby source requires:
```
$ which gcc
/usr/bin/gcc-4.2
```

If this command does not show a path to gcc-4.2, you'll need to install
the older OSX GCC compiler.  *Check to see if Xcode is installed first!*
Running the osx-gcc-installer while Xcode is installed is known to cause
various issues.  Uninstalling any recent version Xcode (version 4.2 or later)
is easy, just delete the Xcode app from the /Applications/ folder.  You can
reinstall it again after you run the osx-gcc-installer.  (If you have
an older version of Xcode installed, then you don't need install the older
OSX GCC gcc-4.2 compiler, and thus, don't need to remove Xcode [or run the
osx-gcc-installer]).

Download the [OSX GCC Installer](https://github.com/kennethreitz/osx-gcc-installer/downloads).
For Mac OS X Lion and Mountain Lion, go with the [GCC-10.7-v2.pkg](https://github.com/downloads/kennethreitz/osx-gcc-installer/GCC-10.7-v2.pkg)

After the osx-gcc-installer completes, reinstall Xcode again.  Also install
the command line tools, by clicking on the "Preferences" menu, then the
"Downloads" preference pane to install the "Command Line Tools".

After gcc-4.2 is installed, download the rbenv source distribution to
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

And remember to remove whatever you add to <code>$HOME/.bash_profile</code>
