---
date: 2008-03-23 04:41:48+00:00
slug: take-ownership-of-files-and-folders-through-script
title: Take ownership of files and folders through script
tags:
- .net
- howto
- powershell
- scripting
- server 2003
- windows
- windows server
---

As part of our process to disable user accounts, we take ownership of the user's server-stored documents such as roaming profiles and redirected My Documents directories. We then either keep access restricted to the domain admins group or grant access to a replacement user who should receive access to the departed user's files.

With an upgrade to Exchange 2007, we have taken advantage of the Powershell access to Exchange objects, and have scripted the mailbox provisioning and account disable processes. One of the sticking points in getting the disable script wrapped up was seizing control of the user's directories. Now, Powershell does have the ability to modify ACL's through the New-Acl and Set-Acl cmdlets (links below), but the users have exclusive access to their server-side directories. It is easy enough to take ownership of a directory through the Windows Explorer Security dialog, but the Powershell methods all presented some form of error when trying to set permissions or change ownership on a file system object to which you do not already have access to.

<!-- more -->

I struggled for some time off and on to try to work around this with a native Powershell way of seizing control of a directory, but I simply could not find what I was looking for. Eventually, I fell back to a simple tool built into Server 2003 already: Takeown.exe. Through a simple line, takeown got me the results I wanted. I built an array of strings for the directories I wanted to take ownership of, generally in a UNC path such as \\servername\users$\[sAMAccountName], then wrapped the takeown line in a Foreach loop:

```
Foreach ( $directory in $directories )
{
takeown.exe /A /R /D Y /F $directory
}
```

To learn more about the options for Takeown, simply type Takeown /? at the command line. For reference:



	
  * /A - Grants ownership to the Administrators group rather than a particular user.

	
  * /R - Recurses

	
  * /D Y - Sets the default to prompts to _Yes_

	
  * /F - The file name of the file system object to take ownership of


After taking ownership, the regular Powershell native cmdlets can be used to set up permissions as are required. For more information on Powershell ACL tools, check out the following links:

	
  * [Play with ACL in MSH](http://mshforfun.blogspot.com/2005/12/play-with-acl-in-msh.html)

	
  * [PowerShell: Set-Acl Does Not Appear to Work](http://blog.netnerds.net/2007/07/powershell-set-acl-does-not-appear-to-work/)

	
  * [File system: Allow inheritable permissions from parent to propagate](http://richardsiddaway.spaces.live.com/Blog/cns!43CFA46A74CF3E96!1069.entry)


Bitcoin tip address for this post: 1MBiHN2jptsRRxYMvjypHH94JhQTv2QGyA
