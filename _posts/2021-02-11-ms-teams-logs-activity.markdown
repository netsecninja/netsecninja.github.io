---
layout: post
title:  "Microsoft Teams Logs for Activity"
date:   2021-02-11
categories: [Analysis]
---
Occasionally, my office is tasked with validating users at-computer activity or lack thereof in response to a security incident, manager suspicions, or possible timecard fraud. Short of having spyware installed on endpoints, we have to look to available logs to help identify when a user was or was not at their computer. In looking for options, I noted a very verbose set of events in the Microsoft Teams logs that can help us in these taskings.

# Log location and retention

I found that by default, the teams logs are located in *C:\Users\\<PROFILE\>\AppData\Roaming\Microsoft\Teams*. They consist of *logs.txt*, and several rollovers named *old\_logs\_\<DATESTAMP\>.txt*. The DATESTAMP is in the format of 4 digit year, 2 digit month, 2 digit day, and finally the local hour, minute, and second of the log rollover. It is unknown what triggers the rollover, as log sizes are very different and each log can contain various counts of days of data. Given that the first line in some rollover logs seems to be cut-off from the previous log file, it's clear the rollover is done on-the-fly with Teams still in operation.

Example:

`old_logs_20210122070635.txt`

The combination of these logs gives you 30 days of information. It is unclear if this retention can be increased by Teams application administrators.

# Log syntax

Each event is prefaced by a datetime stamp in local time for the machine. It's followed by the PID of the Teams instance inside angle brackets. A log type (info, error, etc) is surrounded by double-dashes, followed by the event. 

Example:

`Wed Jan 20 2021 13:20:53 GMT-0700 (Mountain Standard Time) <16408> -- info -- StatusIndicatorStateService: Added Away (current state: NewActivity -> NewActivity)`

# Event types

**Teams app startup**

There are a ton of lines generated on start up that could be used. However, I chose to use this one below as it's short and pretty clear what is happening:

`Fri Jan 22 2021 07:06:50 GMT-0700 (Mountain Standard Time) <16588> -- info -- StatusIndicatorStateService: initialized`

**Teams app shutdown**

If Teams is quit properly by the user, this is the log message you can expect:

`Fri Jan 15 2021 15:00:34 GMT-0700 (Mountain Standard Time) <16308> -- info -- session-end fired`

**Teams app killed**

If Teams crashes or is killed (possibly to install patches?), you will get a different kind of message:

`Thu Jan 21 2021 15:01:51 GMT-0700 (Mountain Standard Time) <16408> -- error -- Child process gone: {"type":"Utility","reason":"killed","exitCode":1073807364,"name":"Network Service"}`

**Machine is locked manually**

If the user locks the machine manually, you will find just this simple message:

`Fri Jan 22 2021 12:30:11 GMT-0700 (Mountain Standard Time) <16588> -- info -- Machine is locked`

**Machine is locked by timeout**

However, if the machine is locked due to an inactivity timeout (whatever your GPO/local settings are), you can expect a message indicating how long the machine was idle, followed by the locked message:

`Wed Jan 20 2021 12:30:21 GMT-0700 (Mountain Standard Time) <16408> -- info -- Machine has been idle for 1198.438 seconds`

`Wed Jan 20 2021 12:31:23 GMT-0700 (Mountain Standard Time) <16408> -- info -- Machine is locked`

I did find that the auto-locking of a machine due to inactivity was not consistent. My company policy is 30 minutes, however, I saw a +/- 5 minute variation in when the machine was actually locked.

**Machine is unlocked**

The machine being unlocked is pretty self-explainatory:

`Fri Jan 22 2021 12:30:18 GMT-0700 (Mountain Standard Time) <16588> -- info -- Machine is unlocked`

**Machine is suspended**

If a user shuts the laptop lid or otherwise suspends the machine, you can expect these messages in succession:

`Mon Jan 18 2021 06:58:43 GMT-0700 (Mountain Standard Time) <16408> -- info -- Machine is locked`

`Mon Jan 18 2021 06:58:44 GMT-0700 (Mountain Standard Time) <16408> -- info -- System has been suspended`

**Away**

Teams indicates when a user is away by changing the icon to a yellow clock. This is also indicated in the logs with the following:

`Wed Jan 20 2021 13:20:49 GMT-0700 (Mountain Standard Time) <16408> -- info -- Machine has been idle for 300.11 seconds`

`Wed Jan 20 2021 13:20:53 GMT-0700 (Mountain Standard Time) <16408> -- info -- userStatusChanged received, so presence is known; and upsEnabledKnown=true`

`Wed Jan 20 2021 13:20:53 GMT-0700 (Mountain Standard Time) <16408> -- info -- StatusIndicatorStateService: Added Away (current state: NewActivity -> NewActivity)`

**Available**

Finally, to indicated when a user is available, the icon is a green checkmark. This is indicated in the logs with the following:

`Wed Jan 20 2021 13:23:22 GMT-0700 (Mountain Standard Time) <16408> -- event -- eventData: s::;m::1;a::4, inactiveTime: 150.437, name: machineState, ppChannel: Production::DC, distSrc: PROPLUS_O365ProPlusRetail, ppInstallMode: UPDATEONLYAPPLY, autoStartPolicy: undefined, vdiMode: 0, eventpdclevel: 2,`

`Wed Jan 20 2021 13:23:23 GMT-0700 (Mountain Standard Time) <16408> -- info -- userStatusChanged received, so presence is known; and upsEnabledKnown=true`

`Wed Jan 20 2021 13:23:23 GMT-0700 (Mountain Standard Time) <16408> -- info -- StatusIndicatorStateService: Added Available (current state: NewActivity -> NewActivity)`

# Parsing the logs

All these logs are great, but it's not easy to summarize the data with just log entries. To assist with this, I created a Teams Activity tool with python to parse the logs, and provide three different outputs. The tool is located [here](https://github.com/netsecninja/teams-activity).

* The Event output shows the datetime stamps of when Teams was started/stopped/killed, when the computer was locked by the user or inactivity, and when it was unlocked.

{: .center}
![Event Output](/assets/Teams-Event_log.png)
* The Activity output shows the datetime stamp and the count of hours of when a user was active at their computer.

{: .center}
![Activity Output](/assets/Teams-Activity_log.png)
* The Daily output shows the sum of the active hours for each day.

{: .center}
![Daily Output](/assets/Teams-Daily_log.png)

**Please note that the times displayed and calculated are only best estimates given the data available.**

# Summary

As you can see, there are some valuable log entries found in the Teams logs. While these were the most notable events found for my use case, there may be others that could provide benefit to your own DFIR investigation and use cases. I would recommend looking into your own Teams logs and seeing what else is available and useful.

