---
date: 2009-02-11 16:24:25+00:00
slug: 550-447-queueexpired-message-expired-when-emailing-mail-enabled-public-folder
title: “550 4.4.7 QUEUE.Expired; message expired” when emailing mail-enabled Public
  folder
tags:
- active directory
- adsi
- exchange 2007
- troubleshooting
- windows
- windows server
---

We’ve been working on some major upgrades to our Exchange environment over the last while. During the course of that, we started receiving NDR’s for messages sent to mail-enabled public folders. Initially, these were “MapiExceptionNotAuthorized” messages, which are related to permissions. Those were sorted out without too much trouble, as the NDR is at least somewhat descriptive. But then we started receiving a very generic NDR of `#550 4.4.7 QUEUE.Expired; message expired ##`.

...not really much to go on. Exchange 2007 does give some more “in plain English, please!” information in its NDR’s, but that also wasn’t much help:

**`Delivery has failed to these recipients or distribution lists:`**

    [user display name]
    Microsoft Exchange has been trying to deliver this message without success
    and has stopped trying. Please try sending this message again, or provide
    the following diagnostic text to your system administrator.

Wow...that was helpful...

<!--more-->

After _much_ digging, I found that the problem stemmed from a public folders server that had been retired. It was going to be re-purposed as an SCR target server, but for now was offline. The trouble was that for the Public Folder tree to which the mail-enabled folder belonged was “owned” by this offline server. This “ownership” is found in the `msExchOwningPFTreeBL `attribute of the public folder store in question. This property is accessible through ADSIEDIT. Now, MS articles to do with this state that the `msExchOwningPFTreeBL` attribute is not directly editable, but you _can_ edit or remove the `msExchOwningPFTree` attribute, which effectively updates the `msExchOwningPFTreeBL` attribute (ref [Public Folder Routing](http://technet.microsoft.com/en-us/library/aa996228.aspx) - Technet). I can’t remember if that’s accurate or not, as this happened some time ago, but at this point I only see the `msExchOwningPFTreeBL` attribute on our PF store, and not the `msExchOwningPFTree` attribute.

For reference, these attributes are on your PF store, which should be located in the Configuration container in ADSIEDIT (`CN=Configuration,DC=yourdomain,DC=[yourtld]`). The full path is:

    CN=Public Folders,CN=Folder Hierarchies,CN=[Your Exchange Administrative Group Name],CN=Administrative Groups,CN=[Your Exchange Organization Name],CN=Microsoft Exchange,CN=Services,CN=Configuration,DC=[yourdomain],DC=[yourtld].

After updating the **msExchOwningPFTree** attribute appropriately to a public folders server that was in production resolved the issue.

Bitcoin tip address for this post:
[16KvZ8hMCNTBhhz9KGZ7MKeW3N1WayaYxR](bitcoin:16KvZ8hMCNTBhhz9KGZ7MKeW3N1WayaYxR)
