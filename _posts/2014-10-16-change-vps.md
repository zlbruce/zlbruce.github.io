---
title: 更换 vps 了
author: zlbruce
layout: post
permalink: /2014/10/16/change-vps/
categories:
  - Blog
tags:
  - Blog
  - DigitalOcean
  - EC2
---
## 更换原因

本来一直使用亚马逊的 EC2，由于我同学弄了个游戏外挂在淘宝上卖，导致流量上涨，而 EC2 的流量是收费的，导致最近每个月的价格都是十几美元，感觉有点贵了。

现在换到了 [DigitalOcean][1]，每月五美元的价格还算承受得起。

## 切换的过程

由于就只用来放一些网站，以及少量的 vpn 需求，切换过程比较简单：

  1. 备份数据库和网页
  2. 拷贝到新 vps 上
  3. 导入数据库和网页
  4. 最后的调整

### 备份

使用 mysqldump 把用到的数据库都备份出来，比如 wordpress, dabr 等

    mysqldump --databases wordpress dabr > mysql_dump.bak

网页的话直接打个包即可

    tar cvfj www_data.tbz2 -C /var/www/ .

### 拷贝

直接使用 scp 命令即可，无需拷贝到本机，以下是在老的 vps 上操作的，其中 do.zlb.me 是新的 vps 的地址，root 为新 vps 的用户名

    scp mysql_dump.bak www_data.tbz2 root@do.zlb.me:/tmp/

### 导入

数据库到导入：

    mysql < /tmp/mysql_dump.bak

网页的导入：

    tar xvf /tmp/www_data.tbz2 -C /var/www/

### 最后的调整

基本上整个迁移就差不多了，不过很有可能新的 vps 上缺少了某些包（比如你的网页需要 php 的 mcrypt, curl 等模块之类的），需要重新安装。或者 apache2 的配置有些不一样（比如说需要启用 ssl, rewrite, headers 等模块），就需要根据实际情况进行调整了。比如我还需要把 SSL 的证书和私钥拷贝到新的 vps 上使用。

最后的最后就是修改 DNS 的指向了，等到 DNS 缓存时间超时后，大家就能访问到新的 vps 了。

## 其他

看了下居然有一年多没有更新过 Blog 了:(，以后要勤奋点，争取多多更新。

 [1]: https://www.digitalocean.com/?refcode=d667079e7273 "DigitalOcean"
