---
date: 2008-08-13 05:51:47+00:00
slug: ssl-errors-in-exchange-2003-public-folder-management
title: SSL Errors in Exchange 2003 Public Folder Management
tags:
- Exchange 2003
- server 2003
- troubleshooting
- windows
- windows server
---

On a recent network audit for a prospective new client, I came across an issue in the Exchange System Manager for their Exchange Server 2003 box. When you tried to browse into any public folder management, ESM presented the following error:

The SSL certificate server name is incorrect.<!-- more -->

I checked the certificate, and it was definitely valid with a trusted Root CA. A quick web search pointed me to [this](http://support.microsoft.com/kb/324345) Microsoft KB. The gist is that whoever set up the Exchange server had set SSL to be required on the \Exadmin virtual directory in the default site on the server. The solution: Clear SSL for the Exadmin vdir and relaunch ESM.

This one is fairly straightforward and seems to be fairly common, but I thought I would throw it up here all the same as I had not really come across it in recent memory.

Cheers,

JaS
