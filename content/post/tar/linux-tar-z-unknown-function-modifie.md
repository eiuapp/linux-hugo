---
date: 2018-09-29T20:00:00+08:00
title: "linux tar: z: unknown function modifie 错误"
weight: 301
menu:
  main:
    parent: "tar"
description : "linux tar: z: unknown function modifie 错误"
---

linux tar: z: unknown function modifie 错误

转载 https://blog.csdn.net/syc001/article/details/72841916

某些linux版本的机器上使用 tar -zxvf *.tar.gz 命令解压.tar.gz时会出现

tar: z: unknown function modifier

错误。

而使用 tar -x *.tar.gz 会出现“tar: /dev/rmt/0: No such file or directory”错误。

 

这是因为该linux下的tar不支持z参数造成的。在这种情况下，可以把解压过程分为两步：

 

gzip -d yourfile.tar.gz。生成一个.tar文件。

tar -xvf yourfile.tar。解压文件。

