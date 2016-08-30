---
date: 2009-02-11 16:24:50+00:00
slug: cannot-install-eset-security-on-debian-40-etch-virtuozzo-vps
title: Cannot install ESET Security on Debian 4.0 Etch Virtuozzo VPS
categories:
- howto
- linux
- nod32
- troubleshooting
---

As part of some mail filter testing, I needed to install ESET Mail Security onto a Debian 4.0 Etch VPS running on Virtuozzo. As a side-note, I found that the install package for ESET’s Gateway Filter, Mail Security, and File Server Security for Linux is all the exact same package; the functionality is basically just controlled/activated by means of licensing the appropriate component.

Anyway, the download comes as an installation script called `esets.i386.deb.bin`. Running that script outputs a license agreement that you have to accept, produces a .deb package called `esets.i386.deb`, and outputs instructions on how to install the .deb package by using dpkg and import the license file. The .deb package installed just fine on another Debian test box, but when I attempted to run `dpkg –i esets.i386.deb` on the Virtuozzo VPS, tar squawked at me that it could not open `/dev/stdin` and the installation bailed:

    hostname:/usr/local/src/eset# dpkg -i esets-3.0.11.i386.deb
    Selecting previously deselected package esets.
    (Reading database ... 24639 files and directories currently installed.)
    Unpacking esets (from esets-3.0.11.i386.deb) ...
    Setting up esets (3.0.11) ...
    Unpacking esets modules ...
    tar: /dev/stdin: Cannot open: No such file or directory
    tar: Error is not recoverable: exiting now
    dpkg: error processing esets (--install):
    subprocess post-installation script returned error exit status 2
    Errors were encountered while processing:
    esets

<!-- more -->

To which I replied, “huh?”

I took a peek around /dev (`ls /dev`), and wouldn’t you know it? No stdin, no stdout, and no stderr! Say what? I checked around ESET’s KB’s, but found nothing related to the issue. I poked around Virtuozzo’s and OpenVZ’s wiki’s, but again came up empty. I checked out the /dev directory (`ls –l /dev/`) on my test box on which I had successfully installed the package, and, as expected, /dev/stdin, /dev/stdout, and /dev/stderr were there. Looking more closely, we see that those files are links to a set of files at /proc/self/fd.

    hostname:~# ls -l /dev/std*
    lrwxrwxrwx 1 root root 15 2009-01-28 02:35 /dev/stderr -> /proc/self/fd/2
    lrwxrwxrwx 1 root root 15 2009-01-28 02:35 /dev/stdin -> /proc/self/fd/0
    lrwxrwxrwx 1 root root 15 2009-01-28 02:35 /dev/stdout -> /proc/self/fd/1
    hostname:~# 

So, on my Debian Virtuozzo VPS, I did the following:

    ln -s /proc/self/fd/0 /dev/stdin
    ln -s /proc/self/fd/1 /dev/stdout
    ln -s /proc/self/fd/2 /dev/stderr 

I re-ran the package installation, and we were set! I don’t know if this issue is particular to some .deb packages or to all, as I had only used `apt-get` up to that point for package installation. As of now, though, I have not yet had any issues with the package or with the links I created…at least not as far as I can tell!

Let me know if you have any other info on this issue.

Bitcoin tip address for this post:
[1PncfTJjuzybqSDcZrP8qJSApHgtZp5B7F](bitcoin:1PncfTJjuzybqSDcZrP8qJSApHgtZp5B7F)
