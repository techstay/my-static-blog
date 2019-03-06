---
title: 更新Manjaro出现Gpgme错误，无效的加密引擎的解决办法
date: 2019-03-06 20:08:24
tags: 疑难杂症
categories: 疑难杂症
---

虽然装了双系统，但是平时我用的主要还是 Windows 系统，另外一个 Manjaro 好久没更新了，这几天忽然想起来，于是赶紧打开更新。果然，滚动式更新时间一长想要在更新的话肯定会出问题。这次遇到的问题是更新到一半图形界面会突然挂掉。前几天更新台式机的时候同样也遇到了这个错误，不过切一下终端，然后在终端里面更新完就好了。没想到笔记本这次遇到问题了，由于平时我设置的 locale 是中文，在笔记本屏幕上还显示成了一堆方块，没办法还得在台式机上 SSH 过去。

笔记本上遇到的问题是“Gpgme 错误，无效的加密引擎”，英文错误信息是“manjaro gpgme error invalid crypto engine”，谷歌一下果然有很多类似信息，这下就方便多了。不过由于我之前瞎试了几次，走了不少弯路。在此直接说解决办法吧。首先这个错误原因我猜测是在图形界面挂掉的时候正好碰到 gpg 这些软件更新，然后导致 gpg 数据库损坏，进而 pacman 无法安装软件。那么解决办法就很明显了，修复 gpg 软件即可。具体可参考[官方文档](https://wiki.manjaro.org/index.php/Pacman_troubleshooting#Errors_about_Keys)。

---

首先删除掉 pacman 原来的可能已经损坏的 key。

```
sudo rm -r /etc/pacman.d/gnupg
```

然后重新安装密钥环，一开始我搜索到的解决方案和这个类似，不过没有 gnupg，所以一直失败。这一点上还是官方文档比较好。

```
sudo pacman -Sy gnupg archlinux-keyring manjaro-keyring
```

然后重新初始化密钥环。

```
sudo pacman-key --init
```

然后加载那些密钥。

```
sudo pacman-key --populate archlinux manjaro
```

最后一步是可选的，删除那些密钥损坏时下载的软件包。

```
sudo pacman -Sc
```

经过这么几步，我终于成功修复了 Manjaro，然后在终端中更新系统并重启之后，终于成功再次进入了 Manjaro 图形界面，真是可喜可贺！
