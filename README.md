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

## Factort Reset Analysis

The conversation between the recovery tool and the device for a failed recovery is as follows:

```
220 ADAM2 FTP Server ready
USER adam2
331 Password required for adam2
PASS adam2
230 User adam2 successfully logged in
SYST
215 AVM EVA Version 1.11581 0x0 0x46409
TYPE I
200 Type set to BINARY
MEDIA SDRAM
200 Media set to MEDIA_SDRAM
P@SW
227 Entering Passive Mode (169,254,115,1,252,99)
RETR env
150 Opening BINARY data connection
226 Transfer complete
USER adam2
331 Password required for adam2
PASS adam2
230 User adam2 successfully logged in
SYST
215 AVM EVA Version 1.11581 0x0 0x46409
TYPE I
200 Type set to BINARY
MEDIA SDRAM
200 Media set to MEDIA_SDRAM
P@SW
227 Entering Passive Mode (169,254,115,1,162,203)
RETR count
150 Opening BINARY data connection
226 Transfer complete
BYE
221 Thank you for using the FTP service on ADAM2
221 Goodbye.
```

```RETR env``` returns the following data:

Note:  None of these credentials are in use so there is no need to redact.  This output is verbatim from WireShark.

```
HardwareFeatures      nand=0xC2F1809502,cpu=0x979
HWRevision            236
HWSubRevision         2
ProductID             Fritz_Box_HW236
SerialNumber          L26270630015393
annex                 A
autoload              yes
bootloaderVersion     1.11581
country               044
crash                 [0]1758420,611c6027,1[1]0,0,0[2]0,0,0[3]0,0,0
firstfreeaddress      0x882540E8
firmware_info         164.07.56
firmware_version      avme
flashsize             nor_size=0MB sflash_size=0KB nand_size=128MB
language              en
linux_fs_start        1
maca                  DC:39:6F:0D:9C:67
macb                  DC:39:6F:0D:9C:68
macwlan               DC:39:6F:0D:9C:69
macwlan2              DC:39:6F:0D:9C:6A
macdsl                DC:39:6F:0D:9C:64
memsize               0x10000000
mtd0                  0x0,0x2C00000
mtd1                  0xB00000,0xF00000
mtd2                  0x0,0x2C0000
mtd3                  0x2C0000,0xB00000
mtd4                  0xF00000,0x1300000
mtd5                  0x1300000,0x8000000
my_ipaddress          169.254.115.1
prompt                Eva_AVM
provider              2018-02-01_zen
tr069_passphrase      BpbrLxCUc25b
tr069_serial          00040E-DC396F0D9C67
usb_board_mac         DC:39:6F:0D:9C:65
usb_device_id         0x0000
usb_device_name       USB DSL Device
usb_manufacturer_name  AVM
usb_revision_id       0x0000
usb_rndis_mac         DC:39:6F:0D:9C:66
webgui_pass           sect9357
wlan_key              89480684294138485085
wlan_ssid             FRITZ!Box#7530#ZV
```

