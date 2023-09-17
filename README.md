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
  
  or
  
- Recovery complete message appears immediately despite doing nothing

It's possible that the tool was getting confused when presented with detecting a 0.0.0.0 IP address, or it was searching on the wrong interface.

I don't have a PC at hand that is not multi-homed or is not running Hyper-V.  A solution that worked for me was to run up a Windows 11 virtual machine and attached just one network adapter.  

Running this machine up meant that Windows was auto-assiging itself a 169.254.0.0/16 IP address.  When the tool starts it sends out an ARP request for a device with a 1 in the last octet, and the device responds.  The tool works 100 percent of the time when run from the virtual machine.

For example:

- Virtual machine address: 169.254.115.123
- Search address: 169.254.115.1

The device boots into the ADAM2 FTP Server (this is true even if the device has been flashed with OpenWrt).

### Aborting the Recovery

The recovery tool would not proceed and presented the following dialogue:

![image](https://github.com/bretmac/amv-fritzboz-7530-openwrt/assets/44399243/b4750056-d81f-45c1-83bf-9760edbeb3f1)


```
Note: The device contains basic settings adapted for your
Internet Service Provider (2018-02_01_zen).  Recovery could
make the device inoperable on the 2018-02_01_zen network.
Please contact your Internet Service Provider instead.
Aborting the recovery.
```
