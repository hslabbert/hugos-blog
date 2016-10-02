+++
date = "2016-09-28T07:17:17-07:00"
description = "A look at BCP38 as a countermeasure to DDoS attacks following the KrebsOnSecurity attacks"
tags = ["infosec", "Internet", "DDoS", "BCP38", "reflection", "security", "networking" ]
title = "What BCP38 Can and Cannot Do"

+++

We're coming through what is seeming like a tipping point in the history of DDoS on the Internet.  Rather than targeting a company or online gaming, one of the largest DDoS attacks ever [targeted an individual](http://krebsonsecurity.com/2016/09/krebsonsecurity-hit-with-record-ddos/), Brian Krebs, most likely for [his work exposing a so-called "booter service"](https://krebsonsecurity.com/2016/09/israeli-online-attack-service-vdos-earned-600000-in-two-years/), a DDoS-for-hire outfit called vDOS, which ultimately led to [the alleged proprietors being arrested](https://krebsonsecurity.com/2016/09/alleged-vdos-proprietors-arrested-in-israel/).

## A brief history of DoS volumes

Public information about DDoS attack volumes are generally sparse outside of news releases and blog posts of DDoS mitigation companies, but even as late as last year, attacks of around 400 Gbps were exceptional events and pretty much the biggest the Internet had seen.  Attacks weighing in at 600 Gbps are said to have been reported in private, but generally haven't hit public knowledge. In the space of a week, we've seen attacks reach those levels and _way_ beyond.  There are reports of _much_ more frequent attacks against common targets, with volumes far above average.  OVH, an international Internet service and hosting provider, reported attacks of over 1 Tbps just the night before the attack on KrebsOnSecurity:

{{< tweet 778019962036314112 >}}

This is an unprecedented escalation in attack volumes and frequency, and this state of affairs cannot go on.  The Internet is *the* platform for discussion in our era, and it is simply unacceptable, whether you are a security researcher, a political activist, someone with an unpopular opinion, or simply someone running a blog for fun, that your voice on the Internet could be silenced by goons with digital cannons because they don't like what you have to say, or simply for the lulz.

After getting [krebsonsecurity.com](https://krebsonsecurity.com) back online on 2016-09-24, Krebs responded to the recent events with a report and call to action in [The Democritizaton of Censorship](https://krebsonsecurity.com/2016/09/the-democratization-of-censorship/).  In it, he calls for a renewed effort on implementing [BCP38]({{< relref "clarifying-ddos-related-terms.md#bcp38" >}}):

> Known as BCP38, its use prevents insecure resources on an ISPs network (hacked servers, computers, routers, DVRs, etc.) from being leveraged in such powerful denial-of-service attacks.

> ...

> BCP38 is designed to filter such spoofed traffic, so that it never even traverses the network of an ISP that’s adopted the anti-spoofing measures. However, there are non-trivial economic reasons that many ISPs fail to adopt this best practice. [This blog post](http://www.internetsociety.org/deploy360/blog/2014/07/anti-spoofing-bcp-38-and-the-tragedy-of-the-commons/) from the Internet Society does a good job of explaining why many ISPs ultimately decide not to implement BCP38.

As we face denial of service attacks, it is important for us to be clear on what tools we have in our arsenal to defend against DDoS attacks.

## No spoofing?  No benefit to BCP38 

I will be very clear that I am a strong proponent of BCP38.  There are objections from some members the network operator community that need to be addressed and solutions to some valid network use cases that need to be developed, but on the whole we should be working to push spoof-detection and mitigation as a default state rather than throwing our hands up in the air and saying it's too hard or a futile effort.

That said: BCP38 does **NOT** defend against direct network attacks that do not use source address spoofing.  Period.

BCP38's sole function is to drop traffic from invalid (spoofed) source addresses.  Now, spoofing _can_ be used in direct attacks in order to provide source address obfuscation, but those attacks need to be "one and done" vectors, e.g. SYN flood attacks or UDP-based attacks that hinge simply on a single packet making its way from the traffic originator (the host we're talking about that's using spoofing) to the target.

A direct, TCP-based attack that requires a TCP 3-way handshake to complete _cannot_ use source address spoofing, as the SYN-ACK back from the target will go to a spoofed address and never reach the attacker to complete the TCP session setup.  As the TCP session setup never completes, any direct attack that requires a valid TCP session will not be able to function.  If source address spoofing is not being used, BCP38 has absolutely **zero** impact on that traffic; it's not spoofed, so BCP38 let's it pass (as it should).

But, the [post-mortem breakdown](http://krebsonsecurity.com/2016/09/krebsonsecurity-hit-with-record-ddos/) from Krebs indicates that these attacks did not use reflection & amplification:

> But according to Akamai, none of the attack methods employed in Tuesday night’s assault on KrebsOnSecurity relied on amplification or reflection. Rather, many were garbage Web attack methods that require a legitimate connection between the attacking host and the target, including SYN, GET and POST floods.

Note the "legitimate connection" part there, which I am reading as a valid TCP session.

Akamai's Martin McKeay also goes on to say that this traffic cannot be spoofed:

> McKeay explained that the source of GRE traffic can’t be spoofed or faked the same way DDoS attackers can spoof DNS traffic. Nor can junk Web-based DDoS attacks like those mentioned above.

Now, I [took some exception]({{< relref "gre-reflection.md">}}) to the statement that GRE traffic cannot be spoofed, and to be fair you can fake a GET or POST as long as you don't care if the TCP session is invalid and the web server at the other end will reject it, but the point here is that the analysis of the attack describes it as consisting of direct attacks including legitimate, completed TCP sessions, _without_ spoofing taking place.

Krebs touches on this again later in the Democritization piece when talking about the IoT devices likely participating in the attack:

> Some readers on Twitter have asked why the attackers would have “burned” so many compromised systems with such an overwhelming force against my little site. After all, they reasoned, the attackers showed their hand in this assault, exposing the Internet addresses of a huge number of compromised devices that might otherwise be used for actual money-making cybercriminal activities...

Again: if they're exposing their IP addresses, that means we're talking about abuse traffic that does _not_ have a spoofed source, which means whether or not BCP38 is implemented on the networks housing the attack sources has **zero** bearing on the effectiveness of the attack.

Krebs does later touch on the possibility of initiatives to require secure-by-default deployments for IoT devices, but that basically amounts to two sentences at the end of the _Shaming the Spoofers_ section.  In light of these attacks being claimed as direct and unspoofed, efforts to secure end hosts would really be the only effective countermeasure, as source address spoofing mitigation has zero effect on traffic that doesn't spoof its source address.

I aim to more about BCP38 in the near future, but I want to state again for emphasis that this post is _not_ a criticism of BCP38 or any kind of statement that BCP38 adoption is pointless or futile.  On the contrary: I wholeheartedly support any efforts to further BCP38 adoption, push for BCP38 adoption on any networks under my control, and believe it is critical to enabling attribution of abuse traffic and mitigation of reflected (and amplified) DDoS attacks.  But, we should not imagine that even getting full BCP38 coverage of the entire Internet will in any way impact network abuse and DDoS attacks that do not use source address spoofing.
