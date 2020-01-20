---
title: gnome桌面共享vnc客户端连接失败需要降低加密级别的解决办法
date: 2018-10-15 00:37:09
tags:
 - 疑难杂症
categories:
 - 疑难杂症
---

由于一些Windows VNC客户端（例如RealVNC、TightVNC等）不支持Gnome桌面Vino服务端的加密，所以会出现这种错误。解决办法是打开dconf编辑器，在org->gnome->desktop->remote-access上，关闭require-encryption选项。

之后再次连接，可以发现这时候就可以连接到Linux桌面啦！

{% asset_img dconf.png dconf编辑器 %}