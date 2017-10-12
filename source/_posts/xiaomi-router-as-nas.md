---
title: Arm the Xiaomi R1D router into a NAS,Remote Downloader,BaiduCloud Syncer
tags:
  - tech
  - openwrt
date: 2016-01-04 14:08:34
---
Xiaomi R1D router is a high performance Broadcom BCM4709 based router with builtin 1TB SATA HDD storage. The hardware of it is really royal except the software. After a painful struggle on the stock firmware of R1D, I decide to give up the kitchen sicker stock firmware.

![](http://www.soyacincau.com/wp-content/uploads/2014/10/141011-xiaomi-mi-router-malaysia-04.jpg)

The stuff we can play with R1D are:

- Flash the Tomato firmware (alternative advanced tomato with better UI)
- Builtin NAS support of tomato with services: USB storage, FTP, samba, DLNA
- Remote access with DDNS support
- Remote offline downloader - Xware downloader from thunder (迅雷)
- Sync files with Baidu cloud storage - syncy.py
- And the released opkg capability of openwrt system for unlimited hacking

The guide can be easily searched with google.

Now I'm very satisfied with the advanced tomato system, thanks to the developers of tomato and advanced tomato system!

![](xiaomi-router-as-nas/QQ20160104-0.png)