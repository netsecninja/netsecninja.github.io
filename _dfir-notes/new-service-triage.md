---
layout: page
title: New Service Triage
permalink: dfir-notes/new-service-traige/
---

Notes from the DFSP episode on New Service Triage

# Windows Event ID
* Security: 4697 "A service was installed in the system"

# Key fields and possible IOCs
* Security ID (User SID)
* Account Name (User who created the service)
* Login ID (Login associated with user who created the service)
  * Look for coorelating Security Event ID 4624 to find login details
* Service Name (Display name)
  * Long names
  * Misspellings
  * Legit sounding names
  * Random names
* Service File Name (executable/script to be run)
  * Usual locations are (anything else is suspect):
    * C:\Windows\System32
    * C:\Program Files
    * C:\Program Files (x86)
* Service Type
  * Suspect values
    * 0x1 (Kernel driver)
    * 0x2 (File system driver)
    * 0x8 (Recognizer driver)
* Service Start Type
  * Suspect values
    * 0x0 (Boot), associated with drivers
    * 0x1 (System), associated with drivers
    * 0x4 (Disabled), uncommon for a new service
* Service Account (user account to run server under)
  * Usually either (anything else is suspect):
    * localSystem
    * localService
    * networkService

# Detection
* Generally, new service installs should be rare
* Low-frequency service names across environment
* Low-frequency file names across environment
* New driver-level services should originate from the driver folder or a legit location

# References
* [DFSP - New Service Triage episode](https://digitalforensicsurvivalpodcast.com/2020/06/23/dfsp-227-new-service-triage/)
* [Windows - Event 4697 Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697)
