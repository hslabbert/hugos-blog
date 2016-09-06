---
date: 2008-01-23 05:13:27+00:00
slug: resize-vhd-files
title: Resize VHD Files
tags:
- .net
- vhd
- virtual server
- virtualization
- windows
---

This topic has been covered a bit ([here](http://blogs.msdn.com/chrisfie/archive/2007/04/19/vhdresizer-a-great-tool-for-resizing-vhd-files.aspx), [here](http://blogs.msdn.com/virtual_pc_guy/archive/2007/03/12/vhd-resizer-resize-and-convert-your-virtual-hard-disks.aspx), and [here](http://autosponge.spaces.live.com/blog/cns!D7F85948C20F0293!247.entry), for instance) but I have been working on a project that utilizes Virtual Server for testing, and it came up again. A consultant that was working on the VM's in question apparently struggled for quite some time before he asked for help on it. So, I thought I would see if another post on this might help someone out.

If you run into a situation where you have existing Microsoft Virtual Server/PC VHD files, but the sizes you created them with initially simply don't cut it anymore, there is hope!

What you will need:



	
  * Original VHD file (obviously)

	
  * [VhdResize](http://vmtoolkit.com/files/folders/converters/entry87.aspx) from [vmtoolkit](http://www.vmtoolkit.com)

	
  * Spare disk space

	
  * .Net 2.0 installed on the machine you will be using for the process


The beauty of the tool is that it can be without having to be installed (self-contained). Just extract the zip, double-click the VhdResize.exe executable, select your source file and destination VHD, and away you go! The VhdResize also allows you to convert from fixed-size VHD's to dynamically-expanding VHD's as well, and it is non-destructive on your source VHD.

Note that this only increases the size of the VHD, so that, effectively, your VM will see a larger physical hard disk present; it does not resize partitions on that drive. For that, you can either use Disk Management or diskpart in your guest VM, or mount the VHD using the _vhdmount_ utility included in Virtual Server and use those disk utilities from your host OS (quick walkthrough [here](http://autosponge.spaces.live.com/blog/cns!D7F85948C20F0293!247.entry)).

Let me know if that's of use!
