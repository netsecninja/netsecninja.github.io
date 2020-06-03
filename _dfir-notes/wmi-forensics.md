---
layout: page
title: WMI Forensics
permalink: dfir-notes/wmi-forensics/
---

Notes from my research into WMI Forensics

# Summary
WMI is a built-in tool that is normal in a Windows environments. Admins, installer scripts, and monitoring software can all use it legitimately. However, WMI can also be used in all attack phases following exploitation. Baseline the normal activity, and look for outliers. As SANS says, "Hunt evil, know normal".

# Terms
* Event Filter - A monitored condition which triggers an Event Consumer
* Event Consumer - A script or executable to run when a filter is triggered
* Binding - Ties the Filter and Consumer together
* CIM Repository - A database that stores WMI class instances, definitions, and namespaces
* MOF - Managed Object Format file, used to define WMI classes to be inserted into the repository
    * Note: these can be named anything and placed anywhere

# Processes
* wmic.exe - Commandline tool for interacting with WMI locally and for remote systems
* wmiprvse.exe - Listening service used on remote systems
* scrcons.exe - SCRipt CONSumer process that spawns child processes to run active script code (vbscript, jscript, etc)
* mofcomp.exe - MOF file compiler which inserts data into the repository
* wsmprovhost.exe - present on remote system if PSRemoting was used

# Files
* C:\Windows\System32\wbem\Repository - Stores the CIM database files
    * OBJECTS.DATA - Objects managed by WMI
    * INDEX.BTR - Index of files imported into OBJECTS.DATA
    * MAPPING\[1-3\].MAP - correlates data in OBJECTS.DATA and INDEX.BTR
* C:\Windows\System32\wbem\AutoRecover - MOF files with `#PRAGMA AUTORECOVER` in first line will be saved here in case the repo needs to be built again, establishing persistence.
    * Review file timestamps

# Registry
* HKLM\SOFTWARE\Microsoft\WBEM\CIMOM\Autorecover MOFs - List of original MOF file locations previously compiled, even if deleted
    * Review file locations and names

# Logs
* Security events
    * 4688 for any of the processes listed above
* Microsoft-Windows-WMI-Activity/Operational
    * 5860 for temporary event consumer creation
    * 5861 for permanent event consumer creation

# Detection of possible abuse
* Privileged login followed by wmic commands
* wmiprvse launching powershell
* Presence of .mof files located outside the C:\Windows\System32\wbem\ directory
* PowerShell invoke-wmimethod, invoke-cimmethod, get-wmiobject, or set-wmiinstance commands
* wmic.exe commands - There are many, but here's some notable ones used in attacks:
    * `process call create` to start processes
    * `/node:` specified target host to run remote commands
* wmiprvse.exe without svchost.exe parent
* wmiprvse.exe with suspect child processes, like powershell
* scrcons.exe is a rare process

# Investigative commands
* Get-WMIObject -Namespace root\Subscription -Class __EventFilter
* Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
* Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding

# Tools
* [FireEye python-cim](https://github.com/fireeye/flare-wmi/tree/master/python-cim)
    * [show_filtertoconsumerbindings.py](https://github.com/fireeye/flare-wmi/blob/master/python-cim/samples/show_filtertoconsumerbindings.py)
    * [timeline.py](https://github.com/fireeye/flare-wmi/blob/master/python-cim/samples/timeline.py)

# References
* [DFSP - WMI Forensics](https://www.youtube.com/watch?v=CgtjINVUpuw)
* [SANS DFIR - Investigating WMI Attacks](https://www.youtube.com/watch?v=aBQ1vEjK6v4)
* [FireEye Windows Management Instrumentation (WMI) Offense, Defense, and Forensics whitepaper](https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/wp-windows-management-instrumentation.pdf)
* [SANS Investigating WMI Attacks blog post](https://www.sans.org/blog/investigating-wmi-attacks/)
