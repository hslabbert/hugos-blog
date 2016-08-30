---
date: 2008-01-23 05:46:35+00:00
slug: exchange-2007-mailbox-guid
title: 'Exchange 2007 Mailbox GUID '
categories:
- active directory
- exchange 2007
- powershell
- scripting
---

On a recent Exchange 2003 to 2007 upgrade, I ran into a very frustrating issue that significantly delayed our deployment. All new mailboxes that were created on using Exchange 2007 tools (Exchange 2007 Management Console or Powershell) were missing several crucial ADSI attributes, namely:



	
  * legacyExchangeDN

	
  * msExchALObjectVersion

	
  * msExchMailboxGuid

	
  * msExchMailboxSecurityDescriptor (set to "not set", all other accounts have a blank value here)

	
  * msExchUserAccountControl


<!-- more -->Of these, the most important seem to be msExchMailboxGuid and msExchMailboxSecurityDescriptor. Without msExchMailboxGuid set, the user account effectively does not have a mailbox. We were desperate enough at one point to create a random mailbox GUID (ensuring first that it is not present anywhere else in the Exchange organization), but the msExchMailboxSecurityDescriptor not being set still ensured that the mailbox was inaccessible.

After a few hours on the phone with MS support, and apparently some contact with a member of the Exchange 2007 development team, we were informed that this was due to a problem with the Exchange System Attendant and something to do with logs...to be honest I was not able to completely understand the guy at this point.

Anyhow, the temporary solution was to restart the System Attendant service on the mailbox server that is experiencing the problem. This is easy enough with Powershell (Restart-Service MSExchangeSA), but we ended up making use of the PsTools suite's psservice because we had multiple mailbox servers going and we needed to sometimes restart the service on a remote mailbox server. Fortunately, restarting the System Attendant service in Exchange 2007 does not restart the Information Store, as was the case with Exchange 2003. After this, the proper attributes will again be stamped and user mailbox provisioning should be successful.

The bug was apparently set to be resolved in SP1 for Exchange 2007 (released late last year), but I have not confirmed this.

Check out these links for more info:

[Arstechnica Forum Thread](http://episteme.arstechnica.com/eve/forums?a=tpc&s=50009562&f=12009443&m=362006418831&r=606003038831#606003038831)
[MSExchange Forums Thread
](http://forums.msexchange.org/m_1800455217/mpage_1/key_/tm.htm#1800455222)[Microsoft Forums (I)](http://forums.microsoft.com/TechNet/ShowPost.aspx?PostID=2020465&SiteID=17&pageid=0)
[Microsoft Forums (II)](http://forums.microsoft.com/technet/ShowPost.aspx?postid=2343802&siteid=17)
