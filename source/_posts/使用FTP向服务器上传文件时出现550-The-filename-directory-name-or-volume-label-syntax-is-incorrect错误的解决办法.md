---
title: >-
  使用FTP向服务器上传文件时出现550 The filename, directory name, or volume label syntax is
  incorrect错误的解决办法
date: 2017-11-15 02:59:01
tags:
 - 疑难杂症
categories:
 - 疑难杂症
---
这种情况一般是由于服务器不支持非ASCII字符（例如中文），解决办法很简单，所有文件都改成英文名称！

顺便说一下，对于自建Linux服务器，可以通过修改locale等方式让服务器支持中文。对于另外一些不太灵活的应用容器（比如Azure），大概就只能老老实实改文件名为英文了。