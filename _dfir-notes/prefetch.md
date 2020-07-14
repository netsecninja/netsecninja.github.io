---
layout: page
title: Windows Prefetch
permalink: dfir-notes/windows-prefetch/
---

# Summary
Windows Prefetch is a background process that monitors roughly the first 10 seconds of an applications execution. It increases the speed of subsequent launches by caching files and resources in memory. Prefetch creates \*.pf files to assist in these launches, but also serves as a valuable forensic artifact. *Important note: Windows workstations have prefetch enabled by default while Windows servers do not.*

# Files
* C:\Windows\Prefetch\\*.pf
* File name is the executable name, followed by a hash of the source files full path and name
* svchost-\*.pf files also includes the command-line parameter in the hashing calculation, hence many versions of scvhost-\*.pf will be found.
* Volume Shadow Copies can contain prefetch files
* Memory may also contain prefectch files

# Artifact details
* Windows Explorer can show the timestamps for creation and modification
* There is a delay of up to 10 seconds between execution and *.pf file creation
* Creation date is the first time the executable was run
* Modification date is the last time the executable was run
* If the dates match, the executable was only run once
* If the dates differ, the executable was run at least twice

# Tools
* [Eric Zimmermans PECMD.exe](https://f001.backblazeb2.com/file/EricZimmermanTools/PECmd.zip)
    * Timestamps
    * Run count and up to the last 7 other run times (total of 8)
    * Shows directories and files referenced
* [Volatility prefetch parser](https://github.com/superponible/volatility-plugins/blob/master/prefetch.py)
    * Prefetch files in memory are compressed on Windows 10 systems, requires [ms-compress](https://github.com/coderforlife/ms-compress) library loaded on *nix analysis systems
        * Download, build, and move the created *.so file to /usr/lib

# References
* [13 Cubed - Prefetch Deep Dive](https://youtu.be/f4RAtR_3zcs)
