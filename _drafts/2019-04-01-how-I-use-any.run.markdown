---
layout: post
title:  "How I use Any.Run"
date:   2019-04-01
categories: [How-to]
---
[Any.Run](https://app.any.run) is a relatively new online sandox analysis application that is used to run suspicious executables or visit websites, which records system and network level activity. The creators of this service have provided a free version with tons of great features available. There is a subscription service to unlock even more features, however for my purposes the free version works just fine. In this post I will share the two different ways I use this powerful tool.

## Research

The first way to use Any.Run is for research of threats using the public tasks others have run. In the free version of the tool, all submission results are publically available. This makes Any.Run a very valuable tool for Open Source Intelligence (OSINT). I use these public submissions to identify malware, extract Indicators of Compromise (IOC) and Behaviors of Compromise (BOC), and to identify threat trends. From the homepage click on the icon that looks like a page with a dog eared corner to view the public submissions. This list is updated in realtime as people submit samples for analysis. Use the filter icon to the right of the hash search bar to narrow down your search.

{: .center}
![Any.Run public submissions filters](/assets/any.run-filters.png)

For example, if I see a suspicious domain while monitoring or hunting, I can use the criteria above to look for already public samples. Or if I'm interested in specific malware like Ursnif, that can be found by putting it's name in the tags. However if I am just looking for a general perspective of the threat landscape, I will set the verdict to malicious and start scrolling through the samples from the last day or two. I will look for threat tags I'm intersted in researching and open those in the different browser tab. I like to pull up three to four samples of the same threat family as to get a better idea of any variations.

## Analysis

The second way to use Any.Run is for analysis. If you are unable to find an existing sample, you can submit a URL or file for analysis. Once you create a free account, click on New Task. There is a basic mode which you choose your OS and then either provide a URL or upload a file to be run. An advanced screen is available if you want to make any tweaks such as UAC auto-conform, Anti-evasion, Browser types, and Fakenet or Tor usage. Greyed out options require a subscription.

{: .center}
![Any.Run advanced mode](/assets/any.run-advanced-mode.png)

Once everything is set, click Run. You will then watch a realtime execution of your malware or visit to whatever URL you entered. The session will end by displaying the results of the task. The URL listed in your browser bar can now be saved in your investigation notes or shared out with others. It's important to note that in the free version all tasks run are public tasks, which means everyone else can see the results. **So don't run any malware or visit any websites you think might be targeting to your organization, or might reveal sensitive information.** You also need to be careful of two things: session timeout and user required actions. 

#### Session Timeout

Any.Run by default will only run the virtual machine for 60 seconds. While the analysis is running you may click Add 60s in the top right box to add an additional 60 seconds. You will only have a few times you can extend the time with the free version. This is important as sometime downloads take a while, or malware authors build in a delay to evade automated sandbox analysis engines. Another reason you may want to add time is because you need to perform user actions.

#### User Required Actions

Occasionally the malware or website you are attempting to analyze will require you to perform some actions. Your mouse and keyboard can be used in the VM displayed. For instance, an unzipped file might have several files inside, and you will need to choose which to run. Or a credential phishing site may request your (hopefully) fake creds to proceed past the login screen. Be aware of these types of actions, and be ready to extend the session time if needed.

## Analysis Results

Regardless if you found an existing sample or created your own, you now have a ton of information at your fingertips. While I won't touch on every aspect of the Any.Run result interface, I will explain the parts I use regularly.

#### Info block

{: .center}
![Any.Run Info block](/assets/any.run-info-block.png)

* Sample source, the environment conditions it was run in, and the detected threat tags.
* IOC to list all relevant captured IOCs.
* Option to download the sample.
* A Process graph to show the parent-child relationship of processes observed. Within this display you may click on any item to get further information about it.
* The ATT&CK matrix shows the techniques observed in the sample.

#### Process

{: .center}
![Any.Run Processes and details](/assets/any.run-process-and-details.png)

* The relevant processes are listed with the full command line as parent processes with nested children processes. Icons displayed under the Process ID show behaviors such as network communications, executables launched, etc.
* Clicking on any of these entries shows a detail pane at the bottom with additional information, warnings, and dangers.
* The "More Info" advanced detail screen shows the full command line, as well as system-level actions by this process such as modified files, registry changes, network traffic, etc. For example clicking on a powershell process and then "More Info" might show the Base64 encoded command called, which I demonstrated how to decode in my previous [blog post]({% post_url 2019-03-16-deeper-analysis-through-powershell-decoding %}).

#### Network

{: .center}
![Any.Run Processes and details](/assets/any.run-network-section.png)

* HTTP Requests will show the HTTP result code, the calling process, the full URL, document type, and a few other things. Clicking on one of the entries will show hashes, Exchangeable Image File (EXIF) information, Hex data, as well as a link to download the resulting data.
* Connections shows the connection by protocol, calling process, Domain/IP/ASN info, port, and traffic bytes in each direction. Clicking on one of the entries displays a Hex dump of the packet data of the network session. Here we can see things like HTTP request and response headers, and payload data.
* DNS Requests show the query and responses.
* Threats outputs the alerts fired from Suricata with associated alert details.
* The PCAP icon on the far right allows you to download a packet capture of the sample.

#### Files

{: .center}
![Any.Run Processes and details](/assets/any.run-files-section.png)

* File Modification shows the process, the full path and filename, and file type of any file created or modified.
* Clicking on any of these entries will details of the file including the hash, MIME type, a preview of the contents, and even the option to download a copy.

## Summary

I use this tool daily for analysis and threat research to help build my threat hunting queries and awareness of threats. I realize this blog post really only scratched the surface. Ultimately it has quite an intuitive interface, and I hope I have highlighted enough of the extremly useful features to encourage you to try it out. 
    
