+++
date = "2016-09-12T21:42:27-07:00"
description = "systemd and Ubuntu 16.04 Xenial conspire to bite me in the ass"
tags = ["linux", "networking", "ubuntu"]
slug = "ubuntu-xenial-and-systemd-network-online-target"
title = "Ubuntu Xenial and systemd network online target"

+++

So, I'm bringing a little Chromebox back into use on our home network to do
some basic network services: dhcp, dns, and probably actually using it as a
router (long story; tl;dr TekSavvy says they'll give you a /56 but then the
VDSL modem they sell you can't do DHCPv6-PD properly).

Anyway...

I slapped Ubuntu 16.04 Xenial server on it and started plugging away at a
couple of things.  I opted to use ISC's newer [Kea DHCP
server](https://www.isc.org/kea/) rather than `isc-dhcp-server`.  Little did I
know that Ubuntu and systemd had other ideas...

<!--more-->

After getting Kea set up, I reboot the box at some point, only to have my wife
ask me if I'm screwing around on the network...because her laptop didn't get an
IP after coming back from sleep.

Here I find this gem:

    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.548 INFO  [kea-dhcp4.dhcp4/2384] DHCP4_STARTING Kea DHCPv4 server version 1.0.0 starting
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.550 INFO  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_CFGMGR_ADD_IFACE listening on interface enp1s0
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.550 INFO  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_CFGMGR_SOCKET_TYPE_DEFAULT "dhcp-socket-type" not specified , using default socket type raw
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.554 INFO  [kea-dhcp4.dhcp4/2384] DHCP4_CONFIG_NEW_SUBNET a new subnet has been added to configuration: 10.128.1.0/24 with params: valid-lifetime=4000
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.555 INFO  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_MEMFILE_DB opening memory file lease database: type=memfile universe=4
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.557 INFO  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_MEMFILE_LEASE_FILE_LOAD loading leases from file /var/lib/kea/kea-leases4.csv
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.599 INFO  [kea-dhcp4.dhcp4/2384] DHCP4_CONFIG_COMPLETE DHCPv4 server has completed configuration: added IPv4 subnets: 1; DDNS: disabled
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.600 WARN  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_OPEN_SOCKET_FAIL failed to open socket: the interface enp1s0 is down or has no usable IPv4 addresses config
    Sep 12 19:20:00 cherry kea-dhcp4[2384]: 2016-09-12 19:20:00.600 WARN  [kea-dhcp4.dhcpsrv/2384] DHCPSRV_NO_SOCKETS_OPEN no interface configured to listen to DHCP traffic

Google (and Ubuntu's Launchpad) to the rescue!
[Apparently](https://bugs.launchpad.net/ubuntu/+source/isc-kea/+bug/1550784),
`kea-dhcp4-server.service` doesn't include any references to `network.target`
or `network-online.target` for some reason.  K; add those...no joy.

Try [post
#3](https://bugs.launchpad.net/ubuntu/+source/isc-kea/+bug/1550784/comments/3)
by [halvors](https://launchpad.net/~halvors) in that thread?  No dice...

Trusty ol' [StackExchange
post](http://unix.stackexchange.com/questions/210604/how-to-write-a-systemd-service-unit-file-so-it-waits-until-a-specific-interface)
in an attempt to wait for a specific interface rather than just networking in
general?  Yea no:

    Sep 12 21:37:07 cherry systemd[1]: kea-dhcp4-server.service: Control process exited, code=exited status=203
    Sep 12 21:37:07 cherry systemd[1]: Failed to start ISC KEA IPv4 DHCP daemon.
    Sep 12 21:37:07 cherry systemd[1]: kea-dhcp4-server.service: Unit entered failed state.
    Sep 12 21:37:07 cherry systemd[1]: kea-dhcp4-server.service: Failed with result 'exit-code'.

The fix?  [This beauty](http://unix.stackexchange.com/a/217768) of a wrapper
around an `ifquery` shell script with some sleep` thrown in there for good measure:

    [Unit]
    Description=Wait for all "auto" /etc/network/interfaces to be up for network-online.target
    Documentation=man:interfaces(5) man:ifup(8)
    DefaultDependencies=no
    After=local-fs.target
    Before=network-online.target
    
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    TimeoutStartSec=2min
    ExecStart=/bin/sh -ec '\
      for i in $(ifquery --list --exclude lo --allow auto); do INTERFACES="$INTERFACES$i "; done; \
      [ -n "$INTERFACES" ] || exit 0; \
      while ! ifquery --state $INTERFACES >/dev/null; do sleep 1; done; \
      for i in $INTERFACES; do while [ -e /run/network/ifup-$i.pid ]; do sleep 0.2; done; done'
    
    [Install]
    WantedBy=network-online.target

Throw that beauty in `/etc/systemd/system/ifup-wait-all-auto.service`,
install it with `sudo systemctl enable ifup-wait-all-auto.service`, and then
actually have the `network-online.target` references in your `systemd` unit
definitions work properly.

Ain't progress grand?
