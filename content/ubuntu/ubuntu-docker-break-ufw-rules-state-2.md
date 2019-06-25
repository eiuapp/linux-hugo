---
date: 2018-09-29T20:00:00+08:00
title: "docker break ufw's rules in ubuntu - env2"
weight: 2
menu:
  main:
    parent: "ubuntu"
description : "docker break ufw's rules in ubuntu - env2"
---

docker break ufw's rules in ubuntu - env2

ufw 阻止了从docker容器到外部的网络连接

对我来说这是一个非常标准的设置，我有一台运行docker和ufw的ubuntu机器作为我的防火墙。 如果启用防火墙，则docker实例无法连接到外部

https://blog.36web.rocks/2016/07/08/docker-behind-ufw.html
https://oomake.com/question/4955599

## env ##

当运行docker后

### docker 配置 ###

```bash
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ sudo cat /etc/docker/daemon.json 
{
	"hosts": ["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
	"registry-mirrors": ["https://0d6wdn2y.mirror.aliyuncs.com"],
	"dns" : ["192.168.168.222"]
}
ubuntu@utuntu:~/lcnx/local/lvchuang-server$ sudo cat /etc/default/docker 
# Docker Upstart and SysVinit configuration file

#
# THIS FILE DOES NOT APPLY TO SYSTEMD
#
#   Please see the documentation for "systemd drop-ins":
#   https://docs.docker.com/engine/admin/systemd/
#

# Customize location of Docker binary (especially for development testing).
#DOCKERD="/usr/local/bin/dockerd"

# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"
DOCKER_OPTS="--iptables=false"

# If you need Docker to use an HTTP proxy, it can also be specified here.
#export http_proxy="http://127.0.0.1:3128/"

# This is also a handy place to tweak where Docker's temporary files go.
#export DOCKER_TMPDIR="/mnt/bigdrive/docker-tmp"

ubuntu@utuntu:~/lcnx/local/lvchuang-server$ sudo cat /etc/default/docker | grep DOCKER_OPTS
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"
DOCKER_OPTS="--iptables=false"
ubuntu@utuntu:~/lcnx/local/lvchuang-server$
```

### docker container 现象 ###

```bash
ubuntu@utuntu:~/docker/images/ubuntu$ docker run -it -d --dns 192.168.168.222 --name ubuntu-tools ubuntu-tools:v1.0
b4b3f7cd3d03c09f48eae8b0979678af57a07b2fcf118f80de653f8ef45c4e4e
ubuntu@utuntu:~/docker/images/ubuntu$ docker exec -it ubuntu-tools bash
root@b4b3f7cd3d03:/# cat /etc/resolv.conf 
nameserver 192.168.168.222
root@b4b3f7cd3d03:/# ping qq.com
^C
```

当ufw disable后, 则 container 可以连接外网

```bash
root@b4b3f7cd3d03:/# ping qq.com
PING qq.com (59.37.96.63) 56(84) bytes of data.
64 bytes from 59.37.96.63: icmp_seq=1 ttl=54 time=4.77 ms
64 bytes from 59.37.96.63: icmp_seq=2 ttl=54 time=4.69 ms
^C
--- qq.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.691/4.730/4.770/0.079 ms
root@b4b3f7cd3d03:/# route 
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
root@b4b3f7cd3d03:/# traceroute qq.com
traceroute to qq.com (59.37.96.63), 30 hops max, 60 byte packets
 1  172.17.0.1 (172.17.0.1)  0.095 ms  0.036 ms  0.035 ms
 2  * * *
 3  218.17.137.1 (218.17.137.1)  4.515 ms  5.474 ms  5.535 ms
 4  202.105.103.117 (202.105.103.117)  3.896 ms 202.105.159.205 (202.105.159.205)  3.651 ms 202.105.159.213 (202.105.159.213)  3.763 ms
 5  117.176.37.59.broad.dg.gd.dynamic.163data.com.cn (59.37.176.117)  4.348 ms * *
 6  119.147.223.178 (119.147.223.178)  3.624 ms 119.147.223.254 (119.147.223.254)  3.451 ms 121.34.242.134 (121.34.242.134)  3.370 ms
 7  * * *
 8  14.17.2.250 (14.17.2.250)  6.196 ms 14.17.2.242 (14.17.2.242)  6.363 ms  6.457 ms
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *^C
root@b4b3f7cd3d03:/# 
```

### ufw 日志 ###

显示来自docker的阻塞连接

```bash
$ sudo tail /var/log/ufw.log
Jun 11 11:46:08 utuntu kernel: [68180.673178] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:3c:d0:1e:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:46:08 utuntu kernel: [68180.673448] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:3c:d0:1e:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:46:09 utuntu kernel: [68181.175761] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:17:16:de:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:46:09 utuntu kernel: [68181.176021] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:17:16:de:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:48:14 utuntu kernel: [68306.114435] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:3c:d0:1e:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:48:14 utuntu kernel: [68306.114666] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:3c:d0:1e:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:48:14 utuntu kernel: [68306.617583] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:17:16:de:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:48:14 utuntu kernel: [68306.617832] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:17:16:de:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2  
Jun 11 11:50:19 utuntu kernel: [68431.555632] [UFW BLOCK] IN=enp2s0 OUT= MAC=01:00:5e:00:00:01:78:11:dc:3c:d0:1e:08:00 SRC=0.0.0.0 DST=224.0.0.1 LEN=32 TOS=0x00 PREC=0xC0 TTL=1 ID=0 DF PROTO=2
```


## step ##

### (可跳过)失敗尝试 ###
#### 使用ip添加规则。 ####

```
$ sudo ufw allow in from 172.16.42.2
$ sudo ufw allow out from 172.16.42.2
```

#### ip ####

sudo ufw allow from 172.17.0.0/16 

#### docker0 ####

```bash
sudo ufw allow in on docker0
```

#### /etc/ufw/before.rules ####

```bash
sudo vi /etc/ufw/before.rules
```

编辑/etc/ufw/before.rules如下： 在* filter部分中，在第一个必需行块之后，添加：

```
# docker rules to enable external network access from the container
# forward traffic accross the bridge 
-A ufw-before-forward -i docker0 -j ACCEPT
-A ufw-before-forward -i testbr0 -j ACCEPT
-A ufw-before-forward -m state --state RELATED,ESTABLISHED -j ACCEPT
```
在文件末尾，在显示COMMIT的行之后，添加以下部分：
```
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 172.16.42.0/8 -o eth0 -j MASQUERADE
# don't delete the 'COMMIT' line or these rules won't be processed
COMMIT
```
保存文件后，使用sudo ufw disable && sudo ufw enable重新启动ufw

并没有改变仍然被阻止。 

### 如果您熟悉iptables ###

```bash
sudo ufw show raw
```

但是,我不熟悉,这个就尴尬了.

### /etc/default/ufw以将DEFAULT_FORWARD_POLICY的值更改为"ACCEPT" ###

这时, 参考

https://oomake.com/question/4955599

```
也许这是由于当前版本，但目前的答案不适用于我的系统(Docker 0.7.2与基础Ubuntu映像)。 解决方案解释为here in the official Docker documentation。 对于懒惰的人：

编辑/etc/default/ufw以将DEFAULT_FORWARD_POLICY的值更改为"ACCEPT"，
使用[sudo] ufw reload重新加载。
这可以确保您将流量转发到Docker的桥接网络(就我目前对这些事情的理解而言......)。
```

发现有用.

