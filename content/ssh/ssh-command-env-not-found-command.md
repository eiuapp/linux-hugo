---
date: 2018-09-29T20:00:00+08:00
title: SSH远程命令找不到环境变量
weight: 212
menu:
  main:
    parent: "ssh"
description : "解决SSH远程执行命令找不到环境变量的问题"
---

解决SSH远程执行命令找不到环境变量的问题

## env

- ssh

## 原理

https://blog.csdn.net/whitehack/article/details/51705889 (这个写得最好)
https://www.jianshu.com/p/77ebeb27a2dc (简单)
https://www.cnblogs.com/zhenyuyaodidiao/p/9287497.html

## 实践

在 ssh 服务端 找到 `# If not running interactively, don't do anything` , (在这一行下面,一般会有一句`return`之类的), 在这一行下面, 加入我们需要的环境变量.
但是, 也不能加多了, 加多了, scp 会失效.

如下面,就是加多了, 如果报错, 则导致 远程scp不了文件.

```bash
$ vi /home/lcnx/.bashrc
# If not running interactively, don't do anything
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
nvm use v11.14.0
export PATH=$PATH:/home/lcnx/.nvm/versions/node/v11.14.0/bin
[ -z "$PS1" ] && return
```

应该修改成

```bash
$ vi /home/lcnx/.bashrc
# If not running interactively, don't do anything
export PATH=$PATH:/home/lcnx/.nvm/versions/node/v11.14.0/bin
[ -z "$PS1" ] && return
```

## ssh 客户端 查看所使用的 ssh服务端环境

```
ssh lcnx@120.77.39.189 ". /home/lcnx/.bashrc; env"
ssh lcnx@120.77.39.189 "env"
```

但是下面的 %HOME 取的是 ssh 客户端 的用户,所以无效.

```
ssh lcnx@120.77.39.189 ". $HOME/.bashrc; env"
```
