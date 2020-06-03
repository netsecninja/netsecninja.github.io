---
layout: page
title: Mac Autoruns
permalink: dfir-notes/mac-autoruns/
---

Notes from the DFSP episode on Mac Autoruns

# Per user
* System preferences > Users and Groups. Click on each user you can see the autorun programs for each user.
* /Users/*USER*/Library/LaunchAgents

# For all users
* /Library/LaunchDaemons
* /Library/LaunchAgents
* /Library/StartupItems

# For system applications
* /System/Library/LaunchAgents

# Run as root tools
* /Library/PrivilegedHelperTools

# plist files
* com.apple.*.plist are usually legitimate applications
* Filtered list of processes:

`launchctl list | grep -iv com.apple`

# References:
* [DFSP - Mac Autoruns episode](https://www.youtube.com/watch?v=yMnsVAjHMGE)
* [Vince Tocco - MacOS locations for programs that start on boot – Daemons and Launchctl blog post](https://vincetocco.com/macos-locations-for-programs-that-start-on-boot-daemons-and-launchctl/)
