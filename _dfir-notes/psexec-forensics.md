---
layout: page
title: PSEXEC Forensics
permalink: dfir-notes/psexec-forensics/
---

Notes from the DFSP episode on PSEXEC Forensics

# Source system artifacts
* psexec.exe
* EULA in Registry, software hive
* Shim cache (only written on shutdown. Can use memory to capture)
* AMCACHE
* Prefetch
* MFT File creation date

# Destination system artifacts
* psexecsvc.exe (though using psexec -r will change the service name)
* Event logs: 
  * System Event ID 7045 (new service)
  * Security Event ID 4264 (logon)
  * Security Event ID 4672 (elevation of logon)
  * Security Event ID 4697 (service installed)
  * Security Event ID 5140 (access to a share)
* All child processes are run in session 0
* Named pipes and hidden admin shares used for communication
* Shim cache (only written on shutdown. Can capture memory to analyze)
* AMCACHE
* System hive > CurrentControlSet > Services
* Prefetch
* MFT File creation date

# References
* [DFSP - PSEXEC Forensics episode](https://www.youtube.com/watch?v=a3dnqbAUF1U)
