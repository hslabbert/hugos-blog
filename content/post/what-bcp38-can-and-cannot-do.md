+++
date = "2016-09-28T07:17:17-07:00"
description = ""
draft = true
tags = ["infosec", "Internet", "DDoS", "BCP38", "reflection" ]
title = "What BCP38 Can and Cannot Do"
topics = []

+++

We're coming through what is seeming like a tipping point in the history of DDoS on the Internet.  Rather than targeting a company or online gaming, one of the largest DDoS attacks ever [targeted an individual](http://krebsonsecurity.com/2016/09/krebsonsecurity-hit-with-record-ddos/), Brian Krebs, most likely for [his work exposing a so-called "booter service"](https://krebsonsecurity.com/2016/09/israeli-online-attack-service-vdos-earned-600000-in-two-years/), a DDoS-for-hire outfit called vDOS, which ultimately led to [the alleged proprietors being arrested](https://krebsonsecurity.com/2016/09/alleged-vdos-proprietors-arrested-in-israel/).

### A brief history of DoS volumes

Public information about DDoS attack volumes are generally sparse outside of news releases and blog posts of DDoS mitigation companies, but even as late as last year, attacks of around 400 Gbps were exceptional events and pretty much the biggest the Internet had seen.  Attacks weighing in at 600 Gbps are said to have been reported in private, but generally haven't hit public knowledge. In the space of a week, we've seen attacks reach historic volumes.  There are reports of _much_ more frequent attacks against common targets, with volumes far above average.  OVH, an international Internet service and hosting provider, reported attacks of over 1 Tbps just the night before the attack on KrebsOnSecurity:

{{< tweet 778019962036314112 >}}

This is an unprecedented escalation in attack volumes and frequency, and this state of affairs cannot go on.  The Internet is *the* platform for discussion in our era,, and it is simply unacceptable, whether you are a security researcher, a political activist, or someone with an unpopular opinion, or simply someone running a blog for fun, that your voice on the Internet could be silenced by goons with digital cannons because they don't like what you have to say, or simply for the lulz.

After getting [krebsonsecurity.com](https://krebsonsecurity.com) back online on 2016-09-24, Krebs responded to the recent events with a report and call to action in [The Democritizaton of Censorship](https://krebsonsecurity.com/2016/09/the-democratization-of-censorship/).  In it, he calls for a renewed effort on implementing [BCP38]({{< ref "post/more-about-bcp38.md" >}}):

> Known as BCP38, its use prevents insecure resources on an ISPs network (hacked servers, computers, routers, DVRs, etc.) from being leveraged in such powerful denial-of-service attacks.
> ...
> BCP38 is designed to filter such spoofed traffic, so that it never even traverses the network of an ISP that’s adopted the anti-spoofing measures. However, there are non-trivial economic reasons that many ISPs fail to adopt this best practice. [This blog post](http://www.internetsociety.org/deploy360/blog/2014/07/anti-spoofing-bcp-38-and-the-tragedy-of-the-commons/) from the Internet Society does a good job of explaining why many ISPs ultimately decide not to implement BCP38.

As we face denial of service attacks, it is important for us to be clear on what tools we have in our arsenal to defend against DDoS attacks.  This post looks at BCP38 and how it might apply to the attack against Krebs.<!--more-->
