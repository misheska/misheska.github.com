---
layout: post
title: "Using pik to manage multiple versions of Ruby on Windows"
date: 2013-03-10 18:09
comments: true
categories: ruby
---
Out of the box, Ruby does not provide a mechanism to support multiple installed
versions.  <code>pik</code> is a third-party tool that can be used to switch
between multiple versions of the Ruby interpreter on Windows.

Install Ruby
============
Install at least one Ruby interpreter environment using the RubyInstaller
from [http://rubyinstaller.org](http://rubyinstaller.org).  While pik can
install some versions of Ruby, the versions of Ruby it knows to install is
quite limited, and it is just much easier to use RubyInstaller for Windows
to install them.

The choice of Ruby environments to install is up to you.  I have the latest
version of 1.9.3 installed as my main development environment, and Ruby 2.0.0 
to play around with the next generation of the language.  When you install
Ruby, do not add the Ruby executables to your PATH or associate .rb/.rbw
files with the Ruby installation (the default choice) - <code>pik</code> will
manage this setting:

{% img center /images/rubyinstaller.png 'Ruby Installer' %}

Installing pik
==============
Download the latest .MSI installer from [pik downloads on github](https://github.com/vertiginous/pik/downloads)
(as of this writing [0.3.0](https://github.com/downloads/vertiginous/pik/pik-0.3.0.pre.msi)) and run the Pik Setup Wizard - the default options are fine:
{% img center /images/pikinstaller.png 'image' 'images' %}

You can verify that <code>pik</code> installed properly by running the
following in a Command Prompt:
```
> pik --version
pik 0.3.0.pre on Microsoft Windows [Version 6.1.7601]
by Gordon Thiesfeld (gthiesfeld@gmail.com)
```

Registering Ruby environments with pik
======================================
Use the <code>pik add</code> command to register your installed ruby
environments with pik.  For example, I have Ruby 1.8.7, 1.9.3, and 2.0.0
installed on my Windows development box, so I typed the following in a
Command Prompt:

```
> pik add C:\Ruby187\bin
INFO: Adding:  [ruby-]1.8.7-p371
      Located at:  C:\Ruby187\bin

> pik add C:\Ruby193\bin
INFO: Adding:  [ruby-]1.9.3-p392
      Located at:  C:\Ruby193\bin

> pik add C:\Ruby200\bin
INFO: Adding:  [ruby-]2.0.0-p0
      Located at:  C:\Ruby2000\bin
```

Use the <code>pik list</code> command to list all the ruby interpreters
registered with pik:

```
> pik list
   ruby-1.8.7-p371
   ruby-1.9.3-p392
   ruby-2.0.0-p0
```

Set a default Ruby
==================
The <code>pik use</code> command will allow you to switch between your
registered ruby interpreters:

```
> pik use ruby-2.0.0-p0
> ruby -v
ruby 2.0.0p0 (2013-02-024) [i386-mingw32]
```

Add the <code>--default</code> parameter to set one version as the default:

```
> pik use ruby-1.9.3-p392 --default
```

You can use the <code>pik default</code> command to switch to this version:

```
> pik default
> ruby -v
ruby 1.9.3p392 (2013-02-022) [i386-mingw32]
```

Unfortunately when you open a new command prompt, pik is not automatically
loaded, so you will notice that there is no default ruby loaded:

{% img center /images/rubynewcommand.png 'Ruby not found' %}

Either run the <code>pik default</code> command to load pik (and the
default ruby interpreter) or add one particular Ruby interpreter to your
user or system PATH environment variable (via <code>Control Panel -> System
-> Advanced -> Environment Variables....</code>):

{% img left /images/environment1.png 350 394 'Environment control panel' %}
{% img right /images/environment2.png 350 394 'User PATH' %}
