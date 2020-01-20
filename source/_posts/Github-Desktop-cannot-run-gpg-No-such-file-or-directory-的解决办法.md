---
title: 'Github Desktop cannot run gpg: No such file or directory 的解决办法'
date: 2020-01-21 02:11:54
tags: 疑难杂症
categories: 疑难杂症
---

给Git配置了gpg密钥验证之后，使用Github桌面客户端的时候却出现了如题的错误。其实解决办法也很简单，因为虽然Git Bash自带了gpg工具，但是Github Desktop却不知道啊，所以解决办法很简单，在配置文件里写一下告诉Github客户端就好了。

运行下面命令即可：

```bash
git config --global gpg.program $(which gpg)
```
