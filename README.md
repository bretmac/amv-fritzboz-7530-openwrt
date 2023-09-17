# Running OpenWrt on the AVM Fritz!Box 7530

The objective:  Build a secure network for Home and Home Office with a Wi-Fi mesh.


## Background

I have been using the DrayTek series of SOHO routers successfully for many years now but I've lost a lot of faith in the last few updates, especially what they tried to do when firewalling against localhost.  They are great routers but they simply cannot meet my security objectives (which do go way beyond a 'normal' home / home office user).

It was more than a year ago since I first installed OpenWrt 19.(something) on a Fritz!Box 7530 router.  It was impressive and so much more powerful than the numerous BT routers you still see being sold on eBay with OpenWrt installed that I tinkered around with.

It just gathered dust whilst I was busy with life events and, of course, all the documentation was in my head.   This page is aiming to (re)capture information (and gotchas) that I have found useful, and hope you do to.

Where appropriate I'll include some analysis of the Fritz!Box as well.

The overall solution will be about 5 Fritz!Box 7530 devices to serve as Wireless Access Points and pfSense CE Firewall software running on a spare Dell XPS 8700 PC (i7 4770k, 32GB RAM, 512GB SATA SSD).


## Security Objectives

The world is a very different place than it was in 2021.  Isolation will be a core part of the network security.

- provide a Guest network
- provide egress filtering
- provide OpenVPN access
- isolate most devices using multiple SSIDs and VLANs
- monitoring and logging

The security aspect will be covered more in a separate pfSense documentation repo.


## Factory Reset Fun

One concern was bricking the device.  AVM do provide a utility to reflash a dead device but it was not at all straightforward.

The main issues are:

- Windows-only executable
- It has real problems when running a system that is multi-homed, or possibly with Hyper-V network interfaces.
- It does not reset ISP-configured routers, even after installing OpenWrt on them.

The recovery documentation can be found [here](https://en.avm.de/service/knowledge-base/dok/FRITZ-Box-7530/160_Restoring-the-FRITZ-OS-of-your-FRITZ-Box/)

The recovery tool downloads can be found [here](https://download.avm.de/fritzbox/fritzbox-7530/other/recover/)


### Windows-only executable

There is nothing more to be said, other than to note this here.

### Multi-homed System

In short:  this just does not work for me at all.

The tool did one of two things for me:

- Fritz!Box not found at all despite multiple retries
- Recovery complete message immediately despite doing nothing

