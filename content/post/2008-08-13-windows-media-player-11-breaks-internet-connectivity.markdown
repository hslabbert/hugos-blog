---
date: 2008-08-13 06:18:52+00:00
slug: windows-media-player-11-breaks-internet-connectivity
title: Windows Media Player 11 Breaks Internet Connectivity
tags:
- client apps
- networking
- troubleshooting
- windows
---

Alright, so the title for this post seems pretty out there, but I can guarantee you that I have come across this on multiple machines. I'm not saying "If you install Windows Media Player 11 on your computer, networking will break," I'm just saying that if you experience the symptoms outlined below and you're stuck, trying uninstalling WM11 and the WM11 codec; you just might get lucky.

So, one of the other techs in the office calls me over: He's been beating his head against a wall with a remote user being unable to get internet connectivity on his Windows XP workstation. The tech has been on this thing for hours, tried just about everything he can think of shy of a workstation rebuild, and he's looking for some team support. I have him throw the ticket my way; I figure that another set of eyes can only be helpful. With a bit of digging, we isolate the symptoms:



	
  * Full connectivity to the local server is available

	
  * Name resolution is still solid

	
  * Pings are working to both local and remote addresses

	
  * Anything higher up the stack than pings only work locally, and bail as soon as you cross a router. This includes file shares, RDP, FTP, HTTP/s, MAPI, and I'm guessing anything else higher than layer 4.<!-- more -->


About an hour or so later, I'm still not much farther than tech #1. I've reset the stack (`netsh winsock reset`) and rebooted, updated/reinstalled the network card drivers...heck, we even flashed the BIOS on the thing. Nothing...zip...nada.

So, I start thinking that maybe some weird malware/adware that's intercepting traffic and messing things up. I head to Add/Remove Programs and start flipping through the titles. Nothing nasty stands out, but I remove the IE toolbars just to be safe as I've had some personal experience with Yahoo! toolbars causing crashes and other wonderful things in IE. Still no joy...

I get to the end of the list and see an entry for **Windows Media Player 11 codec**. Now, fortunate for me, I don't play around with WM much, and so I'm not sure if this is a legit piece of software or not. What I do remember is a particularly nasty experience I had with some malware hiding in the Add/Remove Programs list as media players. So, out goes the codec. Just for good measure, I toss Windows Media Player 11 after the reboot and then reboot the box again. Why? To be honest, I really don't know...I was getting pretty low on bright ideas at that point.

And then, magically, internet connectivity is restored. I still don't have an explanation for this, but for some reason, Windows Media Player 11 being installed on that machine (and at least two others so far in my experience!) caused Layer 5+ routed networking to bail. If you have a KB or something that explains this, please let me know, 'cause that's just messed up!

Anyhow: Happy hunting!

JaS
