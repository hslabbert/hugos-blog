+++
date = "2016-09-22T20:23:52-07:00"
slug = "gre-reflection"
description = "Considering how GRE traffic might be reflected or redirected for DDoS source obfuscation"
tags = ['DDoS', 'GRE', 'reflection', 'Internet', 'networking', 'tech' ]
title = "GRE Reflection?"
+++

Recently, we're seeing an uptick in GRE traffic as part of a DDoS mix.  Most prominently, GRE featured as the biggest volume contributor in the record 600+ Gbps attack on [krebsonsecurity.com](http://krebsonsecurity.com).  (Note that the site is currently offline as it's finding a new home, so any links to krebsonsecurity.com will reference [The Internet Archive](http://archive.org) instead.)

An initial tweet from [@briankrebs](https://twitter.com/briankrebs) listed GRE in the attack traffic mix:

{{< tweet 778402188267778048 >}}

...and Krebs later confirmed more details in KrebsOnSecurity's [own article](https://web.archive.org/web/20160922021000/http://krebsonsecurity.com/2016/09/krebsonsecurity-hit-with-record-ddos/) reporting on the attack:

> Preliminary analysis of the attack traffic suggests that perhaps the biggest chunk of the attack came in the form of traffic designed to look like it was generic routing encapsulation (GRE) data packets, a communication protocol used to establish a direct, point-to-point connection between network nodes.

Now, volumetric DDoS attacks will generally use amplification vectors like open DNS resolvers, misconfigured or vulnerable NTP or SNMP servers, SSDP, etc. in order to boost the attack volume.  Those amplifiers are also often vulnerable to reflection attacks, where the attacker spoofs the source address in the initial amplification trigger packets so that the amplified replies hit the target rather than the attacker.  This can be pulled off because these exploited amplification vectors are stateless and UDP-based, and so a single spoofed packet from the attacker will trigger the amplified reply destined for the target.  A TCP-based attack could yield a larger amplification factor (e.g. just think of pulling off an HTTP GET of a GB+ file!), but a TCP 3-way handshake would never complete successfully with a spoofed source address, and even if somehow it could, the attacker would have to keep ACK-ing somehow in order to keep the transfer going.  GRE we are told, however, could not have a spoofed source address (also from [the same Krebs article](https://web.archive.org/web/20160922021000/http://krebsonsecurity.com/2016/09/krebsonsecurity-hit-with-record-ddos/)):

> McKeay explained that the source of GRE traffic can’t be spoofed or faked the same way DDoS attackers can spoof DNS traffic.

GRE is not known to have an amplification vector, and I haven't been able to think of one.  But is it true that source IPs cannot be spoofed in GRE?  *Note that this is an untested theory and still needs to be validated.*<!--more-->

We need to dig a bit into what GRE actually is in order to test that out.  The tl;dr is [below]({{< ref "#so-how-would-this-be-reflected" >}}).

### Intro to GRE; skip if you know how GRE and keepalives work

Generic Routing Encapsulation, or GRE, is basically a way to wrap up or "encapsulate" a packet inside another packet.  Why would you want to do that?  Most commonly, GRE is used to tunnel traffic across a set of intermediate routers that would not the traffic being tunneled because they don't have a route for its destination.  If you've ever connected to a corporate VPN or used a site-to-site IPSEC VPN, it's pretty much the same idea except GRE does not provide encryption.

Now, GRE tunnels are completely stateless.  This means no agreement needs to made between two routers passing each other GRE traffic about e.g. whether a tunnel is up or down, how many packets have traversed it, anything to do with sequence numbers, etc.  The sending router slaps a GRE header on the packet to be encapsulated, fires it in the direction of the destination router, and calls it a day.  The receiving router similarly just strips off the GRE header and forwards the inner packet if it can.  Now, I'll qualify this by stating that the receiving router can have multiple different GRE tunnels configured, so it should look at the source and destination IP in the GRE header (and possibly a GRE key) in order to figure out which GRE tunnel the packet belongs to, but again this is a stateless operation.

Why would we use GRE rather than something like an IPSEC tunnel?  Well, not all traffic needs network layer privacy.  If we need to somehow tunnel traffic that would be crossing the public Internet in the clear *anyway*, there's no point in encrypting it just because we need to tunnel it.  Ditto if we have traffic for an encrypted protocol (HTTPS, SSH), where we just need to tunnel it to pull routing tricks but don't need to bother encrypting something at the network layer when it has *already* been encrypted.  By skipping encryption and state, GRE is computationally "cheap", so it can be leveraged in places with constrained resources (low end routers forwarding a decent chunk of traffic) or high scale requirements (big fat routers that need to forward a **lot** of traffic in hardware at line rate).

That's all nice, but what does this have to do with traffic reflection?  How can we make some endpoint forward on traffic to our target?

I'll preface this *again* by saying this is thus far just theoretical, and still needs to be labbed up to test it out.  This is very much a hypothesis of "If GRE could be reflected, could this be a possible way?" rather than a statement that "GRE can be reflected, and this is how". 

Now, we may sometimes still want to figure out if the other end of a GRE tunnel is alive and that the tunnel is "up", but there is no keepalive mechanism in the GRE spec.  So, how do we do that?  Well, Cisco, $deity bless 'em, never could pass up an opportunity to invent protocols where there is a dirth of RFC coverage, and so GRE keepalives are born.

GRE keepalives effectively trick the remote router into forwarding the keepalive packet *back* to the sender, all without having to know anything about keepalives itself.  Let's say `RouterA` is the one sending keepalives, and `RouterB` is the other end of the GRE tunnel.

When `RouterA` usually sends traffic through the GRE tunnel, it will take the original packet of user or transit traffic and stuff an additional IP and GRE header onto it.  There are some additional fields available in the GRE header, but effectively it's damn near nothing and just says 'the actual inner packet is *this* protocol'.  That will generally by IPv4 (0x0800) or IPv6 (0x86dd), but really can be any EtherType.  The new IP header has a source IP set to `RouterA`'s own IP address and a destination IP set to `RouterB`'s IP address.  The original user traffic would have looked like this:

     -----------------------------------------------------
    | IP | SRC IP          | DST             | IP PAYLOAD |
    |    | 2001:db8:5::100 | 2001:db8:6::200 |            |
     -----------------------------------------------------

With the additional GRE and IP header added, the encapsulated packet looks like this:

     ----------------------------------------------------------------------------------------------------------------
    | IP | SRC IP        | DST IP        | GRE | GRE           | IP | SRC IP          | DST             | IP PAYLOAD |
    |    | 2001:db8:1::1 | 2001:db8:2::2 |     | PROTOCOL=08dd |    | 2001:db8:5::100 | 2001:db8:6::200 |            |
     ----------------------------------------------------------------------------------------------------------------

Great!  So the GRE packet gets over to `RouterB`, it sees its own IP address as the destination and realizes that it has to do something with this packet rather than forward it on.  The L4 protocol is 47 (GRE), so if it can handle GRE then it realizes it has to check what protocol is next in the stack (e.g. IPv4 as 0x0800 or IPv6 as 0x08dd as described above), strip the GRE header, and then forward the inner packet on its merry way (provided it has a route for the destination, the TTL isn't expired, etc. etc.).

So how does `RouterA` get `RouterB` to forward keepalives back to `RouterA`, especially as we said that `RouterB` doesn't know anything about dem dere fancy GRE keepalives?

`RouterA` actually encapsulates its keepalive packet in GRE **twice**.  The outer headers are what we've discussed here already:  source = `RouterA` IP, destination = `RouterB` IP, and then a GRE header.  The inner packet in this case is not some user traffic that's going to the networks at `RouterB`, but rather **another GRE packet**.  `RouterA` crafts the packet so that the source IP on this *inner* GRE packet is actually *`RouterB`*'s IP address, with the destination set to *its own* IP address (that is, `RouterA`'s IP).  That header stack looks like this:

     --------------------------------------------------------------------------------------------------------------------
    | IP | SRC IP        | DST IP        | GRE | GRE             | IP | SRC IP        | DST IP        | GRE | GRE        |
    |    | 2001:db8:1::1 | 2001:db8:2::2 |     | PROTOCOL=0x08dd |    | 2001:db8:2::2 | 2001:db8:1::1 |     | PROTOCOL=0 |
     --------------------------------------------------------------------------------------------------------------------


For this double-encapsulated packet, things at `RouterB` start off just as described above for regular user traffic.  `RouterB` receives the GRE packet, sees the destination IP is its own, checks the GRE header for the protocol number value and then strips it, and then starts to look at what it should do with the inner payload.  Except now, that inner payload is *another* GRE packet, with a source IP address set to `RouterB`'s own IP and a destination IP set to `RouterA`'s IP.

So what does `RouterB` do?  Well, like a good router, it looks up its route for `RouterA` and dutifully forwards the packet (with the outer IP and GRE header stripped) back over to thattaway.  `RouterA` receives the packet, sees its own IP address as the destination, sees a GRE header, and evaluates that.  GRE keepalives use a special protocol number of 0 (as opposed to 0x0800 or x08dd for IPv4 and IPv6 as mentioned above).  On seeing the protocol field set to 0, `RouterA` realizes this is a GRE keepalive packet, and it resets the GRE tunnel keepalive counter to zero and considers the GRE tunnel to be up.


### So how would this be reflected?

Who says the attacker has to set the destination IP address in the inner GRE packet to their own address?

The very *design* of this is such that `RouterB` in our example does not need to know anything about keepalives or the double encapsulation setup.  It does no correlation or checks between the source and destination IP addresses on the outer IP header and the inner ones.  There is nothing stopping me from setting the destination address to whatever the hell I want.  If I want to hit someone at 2001:db8:ffff::a, I just do:

     -----------------------------------------------------------------------------------------------------------------------
    | IP | SRC IP        | DST IP        | GRE | GRE             | IP | SRC IP        | DST IP           | GRE | GRE        |
    |    | 2001:db8:1::1 | 2001:db8:2::2 |     | PROTOCOL=0x08dd |    | 2001:db8:2::2 | 2001:db8:ffff::a |     | PROTOCOL=0 |
     -----------------------------------------------------------------------------------------------------------------------

To think of it: What's stopping me from stuffing a whole IP packet in there rather than just a minimal-payload GRE keepalive?

     --------------------------------------------------------------------------------------------------------------------------------------
    | IP | SRC IP        | DST IP        | GRE | GRE             | IP | SRC IP        | DST IP           | GRE | GRE             | IPv6    |
    |    | 2001:db8:1::1 | 2001:db8:2::2 |     | PROTOCOL=0x08dd |    | 2001:db8:2::2 | 2001:db8:ffff::a |     | PROTOCOL=0x08dd | PAYLOAD |
     --------------------------------------------------------------------------------------------------------------------------------------

I see two possible permutations of this, with varying levels of difficulty and probability of execution:

`1.` Targeted to configured GRE tunnels, with source address spoofing

Routers processing GRE traffic *should* generally first check if they have a tunnel configured with matching source and destination IPs.  If I'm `RouterB`, I should first check that I have a GRE tunnel configured towards `RouterA`'s IP before I process GRE packets with source IPs of `RouterA` destined for me.  Provided routers are doing that correctly, this means an attack would have to (a) know about (or guess / scan for) configured GRE tunnels on potential reflectors in order to pull this off and (b) successfully spoof `RouterA`'s source IP.

A search could theoretically be possible by sending a GRE-encapsulated packet towards `RouterB` with an IP payload destined for an IP that the attacker controls, with the source IP on the outer IP header being varied as the IPs being tested.  The attacker could then listen for that payload packet, and if received, confirm that the tested source IP does in fact have a GRE tunnel configured with `RouterB`.  This is a massive search space, though, as each potential reflector would need to be tested against each other potential source IP on the public Internet.  Even in the IPv4 Internet, this is huge, as you're dealing with `(2**32) * (2**32 - 1)` combinations (minus RFC1918, multicast, documentation, and other reserved ranges).  That's 18,446,744,069,414,584,320 combinations, for those keeping score at home.  Unless the attackers have prior knowledge about potential reflectors and configured GRE tunnels, this possibility is somewhat remote.

Second to that, this requires that the attacker be able to spoof `RouterA`'s IP address.  This is less of a problem given that networks that fail to [implement](http://www.bcp38.info/index.php/Main_Page) [BCP38](https://tools.ietf.org/html/bcp38) seem to be plentiful, as we see from the prevalance of reflected traffic we have because of spoofed sources.

`2.`  To any GRE-capable router, but dependent on GRE-processing flaw

GRE has not yet been subjected to the same scrutiny as e.g. DNS and NTP following those becoming heavily abused in reflected and amplified DDoS attacks (how I do not miss the [Christmas of 2013](https://blog.cloudflare.com/technical-details-behind-a-400gbps-ntp-amplification-ddos-attack/)...).  A hypothetical laxity in the processing of GRE traffic could potentially make this a *much* easier vector to exploit for reflection while simultaneously being *very* difficult for origin and reflector networks to combat.

If `RouterB` does *not* validate that it has a GRE tunnel configured with the source IP of the GRE packet, the attacker could send a GRE packet to `RouterB` *simply from his own IP address or any other IP they want* and still have `RouterB` process the GRE packet, strip the header, and pass on the inner packet to the target.  This *drastically* reduces the search space on the IPv4 Internet from 18,446,744,069,414,584,320 down to `2**32` or roughly 4 billion.  It also means the attacker *does not even need to spoof the target's IP address* in order to pull of the reflection attack, as the target IP is contained *within* the encapsulated packet.  They can opt to spoof their source IP if they have access to a network that does not implement BCP38, but this would be an optional step that provides additional obfuscation and shielding from attribution and traceback.

What would make this difficult to combat on the service provider side is that, should such a GRE processing flaw exist, this could be pulled of without any address spoofing, so even networks that properly implement BCP38 would let it pass.  GRE also has perfectly legitimate uses on business and residential networks and so any transiting or hosting networks for the attacker and reflector likely would not be able to simply drop the offending traffic.  Fixes would be dependent on either ACLs or firmware updates on the reflectors.

The chances of this second reflection option are somewhat remote, as having a flaw of this nature could potentially result in a serious NAT traversal flaw as well.  If I can send `RouterB` a GRE-encapsulated packet from any source and have it forward the inner packet along provided it has *any* GRE configured it on, that could smash through NAT.

### Conclusion

This post was largely the result of spitballing how reflection might be possible with GRE traffic in light of the recent spike of GRE in large, volumetric DDoS attacks.  There is no amplification in this, and if anything the attacker needs to send *more* data than reaches the target as they send the payload with 2x GRE/IP header stacks whereas the reflector strips the outer IP & GRE headers and just sends the one, but should it prove possible it could still be useful for obfuscation.

On the plus side, While DDoS targets/victims with complex networks or applications may not be able to drop GRE entirely, simpler targets (e.g. websites) should be able to simply drop any IP protocol 47 (GRE) traffic destined for the site without impact.  Given the traffic volumes we're seeing, that's pretty much completely contingent on having a large scale DDoS scrubbing service in place, but at least the rulesets should be fairly simple provided there aren't actual production GRE tunnels or e.g. PPTP VPNs ([for shame](https://www.schneier.com/academic/pptp/faq.html)!) hiding behind the scrubbing service.  If needed, static GRE tunnels should be easier to pin down in rulesets with specific source and destinations IPs than PPTP VPNs that use GRE and have varied source IPs.

P.S. If this hyptothetical flaw in processing of GRE'd traffic *does* end up being a thing, I call dibs on the "pick a trendy and somewhat obnoxious name for software flaws" rights and dub this **GREflector**!
