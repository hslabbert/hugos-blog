---
date: 2008-03-23 04:03:02+00:00
slug: dell-broadcom-server-2003-sp2-snp-and-toe
title: Dell, Broadcom, Server 2003 SP2 SNP and TOE
tags:
- exchange 2007
- server 2003
- troubleshooting
- windows
- windows server
---

Dell, Broadcom, and Microsoft have decided to partner up with the release of a technology called TCP/IP Offloading, or TOE for _TCP/IP Offload Engine_. It was bundled together in the Scalable Network Pack (SNP), included and enabled by default with Service Pack 2 (SP2) for Windows Server 2003. The gist of this technology is to enable high-load enterprise applications to be easily scalable. For those of you familiar with the OSI model, TOE moves layer 3 and 4 processing out of the OS and CPU into the NIC. The idea is to better utilize advances in network card performance and free up CPU cycles for other purposes, such as application-side processing.

This all seems well and good, if they saw fit to properly test the stuff out against their own applications!

<!-- more -->
It started with a few calls: users would get a _**Waiting to Update this Folder**_ message in their Outlook client that did not go away even after several hours, but Outlook still showed as _**Connected to Microsoft Exchange**_. Except this was not just for a few users; every single user on a particular Exchange 2007 mailbox server would be affected...not cool when more than 300 mailboxes from multiple different clients are affected!

Restart system attendant on mailbox server? No joy. Restart  WWW and SSL services on Client Access Server? Wrong again. Restart every freakin' Exchange service on the mailbox and Client Access Server? Still no positive change. Fine! Get the go-ahead and restart the mailbox server? Bingo.

That,  however, is not what we call a _resolution_. At best that's a workaround, but a workaround that comes with guaranteed downtime, and no explanation as to the cause. So we checked the event viewers on the mailbox server and the client access servers, and found...nothing...damn... Run the Best Practice Analyzer for Exchange...nothing useful...damn again... Whatever, chalk it up to freaky coincidences and keep going. That was fine, until it happened again, and again, and again, all with no trace.

Alright, so a call to MS Support it is. Four hours on the phone later, and no real answers. The tech suggests that we [disable TCP/IP Chimney](http://support.microsoft.com/kb/912222), one of the features of the TCP/IP Offloading technology, and we comply. As is to be expected, the support guys wanted to try and pin it on an Outlook issue, as that is where the problem was most visible. But that just doesn't fly when every single client on a particular mailbox server is affected at the exact same time. Anyway, they tell us that they will keep checking on it, but that we will have to call them back when the problem is occurring and _before_ we reboot the affected mailbox server. Okay...well...tell that to the 300+ users screaming for email access they are paying for!

The next time the problem rolls around, I try to get the okay to call MS, but the clients have already had too many email headaches from this problem, and we're forced to reboot the box again to keep people happy. Fortunately, MS gets a higher-up tech in touch with us, and the guy drills us with some useful questions for relevant information. He starts to run through the BPA stuff again and gets us to export the event logs and upload them for further examination. He requests that we download a specific Exchange troubleshooting tool, and upload the logs for that as well. Now we're getting somewhere! Or so I thought.

The mailbox servers that were affected get through the majority of the tests and information gathering of the tool they had us download, but it bails on some RPC-related diagnostics. I report this to the tech, so he directs us to use a different tool that skips around that part. After some scheduled reboots, the original tool completes its diagnostics, and we can upload the results. At this point, we can't afford more problems, so we're actually running _nightly _reboots on the Exchange mailbox and CAS servers. This seems to stave off the problem, but, again, this is just a workaround.

The tech finds nothing in the event viewer to indicate any problems, but he suggests the TCP/IP Chimney disable thing again. This time, however, we also are told to disable any offloading features in the network card device configuration as well. I find the settings and switch them off. We had consolidated mailboxes so that one of the mailbox servers remains with only a few test mailboxes and the mailboxes of some of our support team members on it. It's a gamble, but we remove nightly reboots from that server and switch back to the regular reboot schedule. Weeks pass, and we don't run into any problems. We load some more mailboxes on there, and still no problems.

So, after weeks and weeks of reliability problems, the sneaky culprit was a set of technologies teamed together as the Scalable Network Pack (SNP), introduced without much notice with SP2 for Server 2003. I ended up writing a VB script that determines if a server is running with Broadcom network cards and disables the TOE settings in the TCP/IP stack directly as well as the TOE and SNP settings on the network cards themselves through some registry edits.

I later found that the MSExchange team had posted a [blog entry](http://msexchangeteam.com/archive/2007/07/18/446400.aspx) about potential problems with TOE and Exchange 2007, but had for some reason never come across this post in my searches on the symptoms we were experiencing. I have also since run into posts from other sysadmins reporting that the TOE features cause Internet Security and Acceleration (ISA) Server to lock up completely.

I would post the VB script I created for dealing with this problem, but Microsoft also just released a [hotfix](http://support.microsoft.com/kb/948496) that specifically disables the SNP features. Here is a list of some of the problems the hotfix is meant to address:



	
  * When you try to connect to the server by using a VPN connection, you receive the following error message: Error 800: Unable to establish connection.

	
  * You cannot create a Remote Desktop Protocol (RDP) connection to the server.

	
  * You cannot connect to shares on the server from a computer on the local area network.

	
  * You cannot join a client computer to the domain.

	
  * You cannot connect to the Exchange server from a computer that is running Microsoft Outlook.

	
  * Inactive Outlook connections to the Exchange server may not be cleaned up.

	
  * You experience slow network performance.

	
  * You may experience slow network performance when you communicate with a Windows Vista-based computer.

	
  * You cannot create an outgoing FTP connection from the server.

	
  * The Dynamic Host Configuration Protocol (DHCP) server service crashes.

	
  * You experience slow performance when you log on to the domain.

	
  * Network Address Translation (NAT) clients that are located behind Windows Small Business Server 2003 or Internet Security and Acceleration (ISA) Server experience intermittent connection failures.

	
  * You experience intermittent RPC communications failures.

	
  * The server stops responding.

	
  * The server runs low on nonpaged pool memory


...wow...so pretty much, this _Microsoft_ technology that is turned on _by default_ breaks several _Microsoft_ server products and messes networking almost entirely. Nice...

I hope that this spares somebody some of the pain we had to go through with this.
