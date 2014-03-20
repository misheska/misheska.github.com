---
layout: post
title: "Windows Server 2012 Automated Install Settings"
date: 2013-07-28 20:20
comments: true
categories: 
---

* list element with functor item
{:toc}

_Updated March 19, 2013_

* _Updated ADK link from version 8 to 8.1_
* _Updated link to Autounattend.xml_
* _Added note about using KMS keys_

I just recently revised all my automated install XML files for the Windows
System Preparation Tool (Sysprep) that I use for my Windows development
testbed.  For this go around, I'm documenting the XML answer file settings
for each version of Microsoft Windows.  This article covers the XML answer
files settings needed to automate a Windows Server 2012 (64-bit) base
OS install.

You'll need to use the Windows System Image Manager tool to edit Sysprep
XML answer files.  The Windows System Image Manager is packaged with the
[Windows Assessment and Deployment Kit](http://www.microsoft.com/en-us/download/details.aspx?id=39982)
tool suite.  Download and install the Windows Assessment and Deployment Kit
to install the Windows System Image Manager (WSIM).

Link to [Autounattend.xml](https://raw.githubusercontent.com/misheska/basebox-packer/master/template/windows2012/floppy/win2012-datacenter/Autounattend.xml) with all the settings in this article.  _**NOTE:** Right-click and choose "Download Linked File As..." in your web browser, as many web browsers will try to interpret the Xml_.

Disabling the language settings dialog
======================================

{% img /images/sysprep/win2012x64/language.png %}

{% img /images/sysprep/win2012x64/languagecomponents.png %}

In the _Windows Image_ pane, select the component 
*amd64_Microsoft-Windows-International-Core-WinPE_6.2.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 1 windowsPE_.  Using the
Answer File _Properties_ and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/languagesettings.png %}

* InputLocale = **en-US**
* SystemLocale = **en-US**
* UILanguage = **en-US**
* UserLocale = **en-US**

{% img /images/sysprep/win2012x64/productkeycomponents.png %}

In the _Windows Image_ pane, select the component 
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _UserData/ProductKey_ and choose _Add Setting to Pass 1 windowsPE_.  Using the
Answer File _Properties_ and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/productkeysettings.png %}

* Key = _YOUR_PRODUCT_KEY_
* WillShowUI = **OnError**

The official Microsoft KMS keys are listed [here](http://technet.microsoft.com/en-us/library/jj612867.aspx) and make a good starting point to test installs.

Disabling the Select Operating System dialog
============================================

{% img /images/sysprep/win2012x64/operatingsystem.png %}

{% img /images/sysprep/win2012x64/operatingsystemcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _ImageInstall/OSImage/InstallFrom/Metadata_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/operatingsystemsettings.png %}

* Key = **/IMAGE/NAME**
* Value = **Windows Server 2012 SERVERDATACENTER**

NOTE: Make sure the `/IMAGE/NAME` value matches the Windows Server 2012
Image flavor you selected.  Possible values are:

* Windows Server 2012 SERVERDATACENTER
* Windows Server 2012 SERVERDATACENTERCORE
* Windows Server 2012 SERVERSTANDARD
* Windows Server 2012 SERVERSTANDARDCORE

Disabling the EULA dialog
=========================

{% img /images/sysprep/win2012x64/eula.png %}

{% img /images/sysprep/win2012x64/eulacomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _UserData_ and choose
_Add Setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img /images/sysprep/win2012x64/eulasettings.png %}

* AcceptEula = **true**

Disabling the Disk Allocation dialog
====================================

{% img /images/sysprep/win2012x64/diskallocation.png %}

{% img /images/sysprep/win2012x64/diskconfigurationcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _DiskConfiguration_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/diskconfigurationsettings.png %}

* WillShowUI = **OnError**

{% img /images/sysprep/win2012x64/diskcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _DiskConfiguration/Disk_ and choose
_Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/disksettings.png %}

* DiskID = **0**
* WillWipDisk = **true**

{% img /images/sysprep/win2012x64/createpartitioncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _DiskConfiguration/Disk/CreatePartitions/CreatePartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/createpartitionsettings.png %}

* Extend = **false**
* Order = **1**
* Size = **10000**
* Type = **Primary**

NOTE: Don't worry about getting the size exact - just set it to a
reasonable minimum.  In the next setting, we will extend the partition
to fill all remaining disk space on the drive.

{% img /images/sysprep/win2012x64/modifypartitioncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.2.9200.16384_neutral*,
right-click on _DiskConfiguration/Disk/ModifyPartitions/ModifyPartition_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/modifypartitionsettings.png %}

* Active = **true**
* Extend = **true**
* Format = **NTFS**
* Letter = **C**
* Order = **1**
* PartitionID = **1**

{% img /images/sysprep/win2012x64/installtocomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Setup_6.1.7600.16385_neutral*,
right-click on _ImageInstall/OSImage/InstallTo_ and 
choose _Add setting to Pass 1 windowsPE_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/installtosettings.png %}

* DiskID = **0**
* PartitionID = **1**

Disabling the Administrator password prompt
===========================================

{% img /images/sysprep/win2012x64/setadministratorpassword.png %}

{% img /images/sysprep/win2012x64/setadministratorpasswordcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click on _UserAccounts/AdministratorPassword_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/setadministratorpasswordsettings.png %}

* Value = **vagrant**

Set up vagrant autologin
========================

{% img /images/sysprep/win2012x64/ctrlaltdel.png %}

{% img /images/sysprep/win2012x64/vagrantaccountcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/vagrantaccount.png %}

* Description = **Vagrant User**
* DisplayName = **vagrant**
* Group = **Administrators**
* Name = **vagrant**

{% img /images/sysprep/win2012x64/vagrantpasswordcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click on _UserAccounts/LocalAccounts/LocalAccount/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/vagrantpassword.png %}

* Value = **vagrant**

{% img /images/sysprep/win2012x64/vagrantautologoncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.1.9200.16384_neutral*,
right-click on _AutoLogon_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/vagrantautologon.png %}

* Enabled = **true**
* Username = **vagrant**

{% img /images/sysprep/win2012x64/vagrantimagepasswordcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click on _AutoLogon/Password_ and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_ and
_Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/vagrantpassword.png %}

* Value = **vagrant**

Do not show Server Manager at logon
===================================

{% img /images/sysprep/win2012x64/servermanager.png %}

{% img /images/sysprep/win2012x64/disableservermanageratlogoncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-ServerManager-SvrMgrNc_6.2.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2012x64/disableservermanageratlogonsettings.png %}

* DoNotOpenServerManagerAtLogon = **true**

Disable User Account Control (UAC)
==================================

{% img /images/sysprep/win2012x64/uac.png %}

{% img /images/sysprep/win2012x64/uaccomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-LUA-Settings_6.2.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 2 offlineServicing_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img right /images/sysprep/win2012x64/uacsettings.png %}

* EnableLUA = **false**

Disable Internet Explorer Enhanced Security Configuration
=========================================================

{% img /images/sysprep/win2012x64/ieesc.png %}

{% img /images/sysprep/win2012x64/ieesccomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-ESC_10.0.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img right /images/sysprep/win2012x64/ieescsettings.png %}

* IEHardenAdmin = **false**
* IEHardenUser = **false**

Disable Internet Explorer First Run Wizard
==========================================

{% img /images/sysprep/win2012x64/iefirstrun.png %}

{% img /images/sysprep/win2012x64/iefirstruncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_10.0.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2012x64/iefirstrunsettings.png %}

* DisableAccelerators = **true**
* DisableFirstRunWizard = **true**
* Home_Page = **about:blank**

{% img /images/sysprep/win2012x64/iefirstrunwowcomponents.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_10.0.9200.16384_neutral*,
right-click and choose _Add Setting to Pass 4 specialize_.  Using the 
Answer File _Properties_ and _Settings_ panes, configure the following
settings:

{% img /images/sysprep/win2012x64/iefirstrunsettings.png %}

* DisableAccelerators = **true**
* DisableFirstRunWizard = **true**
* Home_Page = **about:blank**

Replace Internet Explorer Bing search with Google
=================================================

{% img /images/sysprep/win2012x64/googlesearchcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-IE-InternetExplorer_10.0.9200.16384_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/googlesearchsettings.png %}

* ScopeDefault = **true**
* ScopeDisplayName = **Google**
* ScopeKey = **Google**
* ScopeUrl = **http://www.google.com/search?q={searchTerms}**
* ShowSearchSuggestions = **true**

{% img /images/sysprep/win2012x64/googlesearchwowcomponents.png %}

In the _Windows Image_ pane, select the component
*wow64_Microsoft-Windows-IE-InternetExplorer_10.0.9200.16384_neutral*,
right-click on _SearchScopes/Scope_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/googlesearchsettings.png %}

* ScopeDefault = **true**
* ScopeDisplayName = **Google**
* ScopeKey = **Google**
* ScopeUrl = **http://www.google.com/search?q={searchTerms}**
* ShowSearchSuggestions = **true

Enable Remote Desktop
=====================

{% img /images/sysprep/win2012x64/tsconnectioncomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-TerminalServices-LocalSessionManager_6.2.9200.16384_neutral*,
right-click and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/tsconnectionsettings.png %}

* fDenyTSConnections = **false**

{% img /images/sysprep/win2012x64/firewallgroupcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Networking-MPSSVC-Svc_6.2.9200.16384_neutral*,
right-click on _FirewallGroups/FirewallGroup_ and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2012x64/firewallgroupsettings.png %}

* Active = **true**
* Group = **Remote Desktop**
* Key = **RemoteDesktop**
* Profile = **all**

{% img /images/sysprep/win2012x64/rdpsecuritycomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-TerminalServices-RDP-WinStationExtensions_6.2.9200.16384_neutral*,
right-click and choose
_Add Setting to Pass 4 specialize_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img right /images/sysprep/win2012x64/rdpsecuritysettings.png %}

* SecurityLayer = **1**
* UserAuthentication = **0**

Turn off computer password
==========================

Prevent the image from changing its computer account password,
so you can restore old snapshots without being dropped from a domain

    REG ADD "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" /v DisablePasswordChange /t REG_DWORD /d 1 /f

{% img /images/sysprep/win2012x64/disablepasswordchangecomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click _FirstLogonCommands/SynchronousCommand" and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2012x64/disablepasswordchangesettings.png %}

* CommandLine = **REG ADD "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" /v DisablePasswordChange /t REG_DWORD /d 2 /f**
* Description = **Disable computer password change**
* Order = **1**
* RequiresUserInput **true**


Turn off all power saving and timeouts
======================================

{% codeblock set-power-config.bat %}
REM Set power configuration to High Performance
powercfg -setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
REM Monitor timeout
powercfg -Change -monitor-timeout-ac 0
powercfg -Change -monitor-timeout-dc 0
{% endcodeblock %}

{% img /images/sysprep/win2012x64/powerconfigcomponents.png %}

In the _Windows Image_ pane, select the component
*amd64_Microsoft-Windows-Shell-Setup_6.2.9200.16384_neutral*,
right-click _FirstLogonCommands/SynchronousCommand" and choose
_Add Setting to Pass 7 oobeSystem_.  Using the Answer File _Properties_
and _Settings_ panes, configure the following settings:

{% img /images/sysprep/win2012x64/powerconfigsettings.png %}

* CommandLine = **cmd.exe /c a:set-power-config.bat**
* Description = **Turn off all power saving and timeouts**
* Order = **2**
* RequiresUserInput = **true**

