---
layout: post
title: "Windows Server 2008 R2 automated install settings"
date: 2013-07-25 23:20
comments: true
categories: 
---
* list element with functor item
{:toc}

I just recently revised all my automated install XML files for the Windows
System Preparation Tool (Sysprep) that I use for my Windows development
testbed.  For this go around, I'm documenting the XML answer file settings
for each version of Microsoft Windows.  This article covers the XML answer
files settings needed to automate a Windows Server 2008 R2 (64-bit) base
OS install.

You'll need to use the Windows System Image Manager tool to edit Sysprep
XML answer files.  The Windows System Image Manager is packaged with the
[Windows Assessment and Deployment Kit](http://www.microsoft.com/en-us/download/details.aspx?id=30652)
tool suite.  Download and install the Windows Assessment and Deployment Kit
to install the Windows System Image Manager (WSIM).

Disabling the language settings dialog
======================================

{% img /images/sysprep/win2008R2x64/language.png %}

In the _Windows Image_ pane, select the component 
*amd64_Microsoft-Windows-International-Core-WinPE_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 1 windowsPE_.  Using the
Answer File _Properties_ and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/languagesettings.png %}

Disabling the Select Operating System dialog
============================================

{% img /images/sysprep/win2008R2x64/operatingsystem.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _ImageInstall/OSImage/InstallFrom/Metadata_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/operatingsystemsettings.png %}

Disabling the EULA dialog
=========================

{% img /images/sysprep/win2008R2x64/eula.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _UserData_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/eulasettings.png %}

Disabling the Disk Allocation dialog
====================================

{% img /images/sysprep/win2008R2x64/diskallocation.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/diskconfigurationsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/disksettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk/CreatePartitions/CreatePartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/createpartitionsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk/ModifyPartitions/ModifyPartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/modifypartitionsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _ImageInstall/OSImage/InstallTo_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/installtosettings.png %}

Disabling the Administrator password prompt
===========================================

{% img /images/sysprep/win2008R2x64/setadministratorpassword.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _UserAccounts/AdministratorPassword_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/setadministratorpasswordsettings.png %}

Set up vagrant autologin
========================

{% img /images/sysprep/win2008R2x64/ctrlaltdel.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/vagrantaccount.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/vagrantpassword.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _AutoLogon_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/vagrantautologon.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _AutoLogon/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/vagrantpassword.png %}

Disable Initial Configuration Dialog
====================================

{% img /images/sysprep/win2008R2x64/initialconfiguration.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-OutOfBoxExperience_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/initialconfigurationsettings.png %}

Do not show Server Manager at logon
===================================

{% img /images/sysprep/win2008R2x64/disableservermanageratlogon.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-ServerManager-SvrMgrNc_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/disableservermanageratlogonsettings.png %}

Disable User Account Control (UAC)
==================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-LUA-Settings_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 2 offlineServicing_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/uacsettings.png %}

Disable Internet Explorer Enhanced Security Configuration
=========================================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-ESC_8.0.7601.17514_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/ieescsettings.png %}

Disable Internet Explorer First Run Wizard
==========================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_8.0.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/iefirstrunsettings.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_8.0.7601.17514_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2008R2x64/iefirstrunsettings.png %}

Replace Internet Explorer Bing search with Google
=================================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_8.0.7600.16385_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/googlesearchsettings.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_8.0.7601.17514_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2008R2x64/googlesearchsettings.png %}
