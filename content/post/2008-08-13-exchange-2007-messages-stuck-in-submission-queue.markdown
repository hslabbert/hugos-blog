---
date: 2008-08-13 04:32:24+00:00
slug: exchange-2007-messages-stuck-in-submission-queue
title: 'Exchange 2007: Messages stuck in Submission Queue'
categories:
- exchange 2007
- network management
- server 2003
- troubleshooting
- windows
- windows server
---

We recently received reports of message delivery delays in our Exchange organization. We run Exchange 2007, so I checked out the Hub Transport Servers and discovered that messages were piling up in the **Submission** queues on both of the main hub transports. Restarting the Microsoft Exchange Transport service didn't get things going again, so I turned to the Application Log to try to figure out what was going on.<!-- more -->

I noticed some errors coming from the source **MSExchange Extensibility**, with an event ID of 1050. They read as follows:


<blockquote>The execution time of agent 'ScanMail Routing Agent' exceeded 300000 (milliseconds) while handling event 'OnSubmittedMessage'. This is an unusual amount of time for an agent to process a single event. However, Transport will continue processing this message.</blockquote>


From the event, you can tell we're using TrendMicro ScanMail as part of our edge protection. It looked like the ScanMail scanning agent was stuck on a message and couldn't proceed. So, off to the services console I go. ScanMail has three services installed on the server, including its Master Service. The other services seemed happy, but the Master service timed out when I attempted to stop it. I checked the service properties and found its related executable file and killed it through Task Manager. The service was then recorded in the Services Console as being stopped, and after starting the service, mail started flowing again on that Hub Transport server. I repeated those steps on the second Hub Transport server, and messages came flooding out from there as well.

I checked that ScanMail was up to date on both servers, and they appeared to be. I checked through Trend Micro's site nonetheless, and found that there appeared to be a related update. The links:

List of updates for ScanMail 8:  [http://www.trendmicro.com/download/product.asp?productid=8](http://www.trendmicro.com/download/product.asp?productid=8)
Direct link to the relevant update:  [ScanMail 8.0 for Microsoft Exchange Patch 3](http://www.trendmicro.com/ftp/products/patches/smex_80_win_en_patch3.exe)
The README file:  [http://www.trendmicro.com/ftp/documentation/readme/smex_80_win_en_patch3_Readme.TXT](http://www.trendmicro.com/ftp/documentation/readme/smex_80_win_en_patch3_Readme.TXT)

The relevant portion of the changelog:


<blockquote>21. While ScanMail for Microsoft Exchange 8.0 is running, after a few hours, email messages will start to queue. If user re-starts the ScanMail for Microsoft Exchange services, the application will hang at "Stopping".</blockquote>


I was technically on vacation while this all went down, so I had to hand of the update to someone else to apply, but no one was available for that weekend. The patch was not manually applied on the servers that weekend, but the issue did not return. We've now been running smoothly for about two weeks.

Hopefully you don't run across the issue, and hopefully this post will be of some help if you do!

Bitcoin tip address for this post: 1HC44rd1NUZUiWDsHNf9Y1KPSxbtW7b5Sw
