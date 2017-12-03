---
title: windows安装scrapy出现utf-8 codec can't decode byte的解决办法
date: 2017-12-03 18:24:07
tags:
 - 疑难杂症
categories:
 - 疑难杂症
---

这种情况是由于scrapy的python包是针对linux编写的，而windows下的Powershell终端编码并不是UTF-8，所以出现了这个编码错误。解决办法就是用支持UTF-8的终端，例如Git Bash来运行`pip install scrapy`命令。