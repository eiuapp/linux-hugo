---
date: 2018-09-29T20:00:00+08:00
title: "docker break ufw's rules in ubuntu - env1"
weight: 1
draft: false
menu:
  main:
    parent: "ubuntu"
description : "docker break ufw's rules in ubuntu - env1"
---

docker break ufw's rules in ubuntu - env1

## env ##

当运行docker后

```bash
sudo docker run --detach \
  --restart always \
  --hostname 192.168.168.137 \
  --publish 192.168.168.137:12443:443 --publish 192.168.168.137:80:80 --publish 192.168.168.137:22:22 \
  --name gitlab-ce-11.9.1-2 \
  --volume /srv/gitlab9.1/config:/etc/gitlab \
  --volume /srv/gitlab9.1/logs:/var/log/gitlab \
  --volume /srv/gitlab9.1/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.9.1-ce.0
```
 
也会导致 整个局域网能访问到 80

```bash
sudo docker run --detach \
  --restart always \
  --hostname 192.168.168.137 \
  --publish 127.0.0.1:12443:443 --publish 127.0.0.1:80:80 --publish 127.0.0.1:22:22 \
  --name gitlab-ce-11.9.1-2 \
  --volume /srv/gitlab9.1/config:/etc/gitlab \
  --volume /srv/gitlab9.1/logs:/var/log/gitlab \
  --volume /srv/gitlab9.1/data:/var/opt/gitlab \
  gitlab/gitlab-ce:11.9.1-ce.0
```
  
会导致 无法通过IP `192.168.168.137` 来访问 80

```bash
ubuntu@utuntu:~$ telnet 127.0.0.1 80
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
^]
telnet> 
^CConnection closed by foreign host.
ubuntu@utuntu:~$ telnet 192.168.168.137 80
Trying 192.168.168.137...
telnet: Unable to connect to remote host: Connection refused
ubuntu@utuntu:~$ 
```

## step ##

这时, 参考

https://chaifeng.com/to-fix-ufw-and-docker-security-flaw-without-disabling-iptables/

发现没用.

再参考

https://my.oschina.net/abcfy2/blog/539485
和
https://blog.36web.rocks/2016/07/08/docker-behind-ufw.html

正确.

### 注意 ###

- 无需重启host
- ubuntu下还需要配置 `/lib/systemd/system/docker.service` 的 ExecStart 加上`--iptables=false`, 然后, systemctl daemon-reload` 

### docker ###

编辑/etc/default/docker文件，修改DOCKER_OPTS="--iptables=false"，等同于给docker启动参数添加--iptables=false选项，此选项会禁用docker添加iptables规则。

```
ubuntu@utuntu:~/lcnx/local/lvchuang-admin-server$ sudo cat /etc/default/docker | grep DOCKER_OPTS
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"
DOCKER_OPTS="--iptables=false"
ubuntu@utuntu:~/lcnx/local/lvchuang-admin-server$ 
```

同时,还要修改一下 docker.service 文件

```bash
ubuntu@utuntu:~/lcnx/local/lvchuang-admin-server$ sudo vi /lib/systemd/system/docker.service
ubuntu@utuntu:~/lcnx/local/lvchuang-admin-server$ sudo cat /lib/systemd/system/docker.service | grep ExecStart
ExecStart=/usr/bin/dockerd --iptables=false --containerd=/run/containerd/containerd.sock
ubuntu@utuntu:~/lcnx/local/lvchuang-admin-server$ 
```

重启docker

```bash
sudo systemctl daemon-reload 
sudo systemctl restart docker
```

### ufw ###

编辑/etc/ufw/before.rules这个文件，在文件末尾追加以下内容:

```
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -o docker0 -s 172.17.0.0/16 -j MASQUERADE
COMMIT
```

之后重启一下ufw规则即可:

```
sudo ufw disable
sudo ufw enable
```
