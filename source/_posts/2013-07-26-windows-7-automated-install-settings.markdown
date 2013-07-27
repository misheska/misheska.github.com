---
layout: post
title: "Windows 7 Automated Install Settings"
date: 2013-07-26 10:48
comments: true
categories: 
---

* list element with functor item
{:toc}

I just recently revised all my automated install XML files for the Windows
System Preparation Tool (Sysprep) that I use for my Windows development
testbed.  For this go around, I'm documenting the XML answer file settings
for each version of Microsoft Windows.  This article covers the XML answer
files settings needed to automate a Windows 7 (64-bit) base
OS install.

You'll need to use the Windows System Image Manager tool to edit Sysprep
XML answer files.  The Windows System Image Manager is packaged with the
[Windows Assessment and Deployment Kit](http://www.microsoft.com/en-us/download/details.aspx?id=30652)
tool suite.  Download and install the Windows Assessment and Deployment Kit
to install the Windows System Image Manager (WSIM).

[Settings to Use for an Unattended Installation](http://technet.microsoft.com/en-us/library/dd744272(v=ws.10).aspx)

[Automate Windows Welcome](http://technet.microsoft.com/en-us/library/dd744547(WS.10).aspx)

Link to [Autounattend.xml](https://raw.github.com/misheska/basebox-packer/master/template/misheska-win7x64/floppy/Autounattend.xml)
with all the settings in this article.

Disabling the language settings dialog
======================================

{% img /images/sysprep/win7x64/language.png %}

In the _Windows Image_ pane, select the component 
*amd64_Microsoft-Windows-International-Core-WinPE_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 1 windowsPE_.  Using the
Answer File _Properties_ and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/languagesettings.png %}

Disabling the Select Operating System dialog
============================================

{% img /images/sysprep/win7x64/operatingsystem.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _ImageInstall/OSImage/InstallFrom/Metadata_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/operatingsystemsettings.png %}

Disabling the EULA dialog
=========================

{% img /images/sysprep/win7x64/eula.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _UserData_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/eulasettings.png %}

Disabling the Disk Allocation dialog
====================================

{% img /images/sysprep/win7x64/diskallocation.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/diskconfigurationsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/disksettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk/CreatePartitions/CreatePartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/createpartitionsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _DiskConfiguration/Disk/ModifyPartitions/ModifyPartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/modifypartitionsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _ImageInstall/OSImage/InstallTo_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/installtosettings.png %}

Disabling the account and computer name dialog
==============================================

{% img /images/sysprep/win7x64/account.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/vagrantaccount.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/vagrantpassword.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _AutoLogon_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/vagrantautologon.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _AutoLogon/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/vagrantpassword.png %}

Disable Computer Name dialog
============================

{% img /images/sysprep/win7x64/computername.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/computernamesettings.png %}

Disable Protect Computer dialog
===============================

{% img /images/sysprep/win7x64/protect.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click on _OOBE_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/protectsettings.png %}

Disable User Account Control (UAC)
==================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-LUA-Settings_6.1.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 2 offlineServicing_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win7x64/uacsettings.png %}

Disable Internet Explorer First Run Wizard
==========================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_8.0.7600.16385_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win7x64/iefirstrunsettings.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_8.0.7601.17514_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win7x64/iefirstrunsettings.png %}

Replace Internet Explorer Bing search with Google
=================================================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_8.0.7600.16385_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/googlesearchsettings.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_8.0.7601.17514_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/googlesearchsettings.png %}

Enable Remote Desktop
=====================

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-TerminalServices-LocalSessionManager_6.1.7601.17514_neutral*,
right-click and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/tsconnectionsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Networking-MPSSVC-Svc_6.1.7601.175414_neutral*,
right-click on _FirewallGroups/FirewallGroup_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/firewallgroupsettings.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-TerminalServices-RDP-WinStationExtensions_6.1.7601.17514_neutral*,
right-click and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/rdpsecuritysettings.png %}

(Really) Disable Set Network Location Prompt
============================================

{% img /images/sysprep/win7x64/networklocation.png %}

    REG ADD "HKLM\System\CurrentControlSet\Control\Network\NewNetworkWindowOff"

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click _FirstLogonCommands/SynchronousCommand" and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/networklocationsettings.png %}

Turn off computer password
==========================

Prevent the image from changing its computer account password,
so you can restore old snapshots without being dropped from a domain

    REG ADD "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" /v DisablePasswordChange /t REG_DWORD /d 1 /f

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click _FirstLogonCommands/SynchronousCommand" and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/disablepassworchangesettings.png %}


Turn off all power saving and timeouts
======================================

    REM Set power configuration to High Performance
    powercfg -setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
    REM Monitor timeout
    powercfg -Change -monitor-timeout-ac 0
    powercfg -Change -monitor-timeout-dc 0

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.7601.17514_neutral*,
right-click _FirstLogonCommands/SynchronousCommand" and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win7x64/powerconfigsettings.png %}
