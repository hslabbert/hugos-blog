---
date: 2008-03-30 19:12:52+00:00
slug: vista-offline-files-and-smb-opportunistic-locks
title: Vista Offline Files and SMB Opportunistic Locks
tags:
- file management
- troubleshooting
- vista
- windows
---

One of our techs recently ran across a problem with a new Windows Vista Business laptop trying to synchronize offline files to a Windows Server 2000 file server. Synchronization would start, but the Sync Center in Vista would show failures for every single file that was attempted to be sync'd. The error message read something to the extent of "_The process cannot access the file because it is being used by another process_".

We tried the usual: checking permissions on the folders being offline'd (I know that's probably not a word, but you get what I mean); deleting his local cache of Offline Files; disabling and then re-enabling Offline Files. But we just kept on banging our heads against the same error. At first, just about any web search for the error resulted in either something about Windows Home Server or databases or something of the like.  Eventually, though, we struck gold:

[http://support.microsoft.com/kb/296264/en-us](http://support.microsoft.com/kb/296264/en-us): Configuring opportunistic locking in Windows

<!-- more -->

According to MS, opportunistic locking "_lets clients lock 		  files and locally cache information without the risk of another user changing 		  the file_". Opportunistic Locking is a feature of SMB. It is enabled by default, and is configurable from the client side (i.e. request opportunistic locking or not) and on the server side (i.e. allow opportunistic locking on local files or not).

Also note: "_If you disable opportunistic locking, the offline files feature in Windows 		  Vista fails_"

So, apparently, "_The process cannot access the file because it is being used by another process" _actually means_ "Opportunistic locking is not enabled on the file server_". Nice...

Be aware that this setting is changed through the registry, so the regular warnings about messing about in REGEDIT apply (read [Disclaimer](http://bamboo.slabnet.com/~hslabbert/blog/disclaimer/)). That stuff is already listed in the KB, so I won't bother to repeat it.

To enable opportunistic locking on the file server, open REGEDIT to HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters, and change the value of the REG_DWORD
**EnableOplocks** from 0 to 1.

The server does need to be rebooted in order for the registry change to take effect. After rebooting, retry your offline files synchronization. If the opportunistic locking was your problem, you should be good to go!

UPDATE:

I must also thank Rainier Day for [his post](http://blog.rainiernetworks.com/2008/06/25/vista-synchronization-errors/) on his struggles with Opportunistic Locking. He was hit from the other side: The server-side opslock settings were good, but his Vista clients were configured to _not_ request opportunistic locks, and this caused the clients to also experience offline files synchronization errors. Thanks Ranier: I noticed that component in the MS KB, but had never actually come across a Vista workstation that was configured out-of-the-box to have OpsLocks disabled!

Cheers,

JaS
