---
title: 使用 vps 搭建 openvpn
author: zlbruce
layout: post
permalink: /2011/07/08/vps-openvpn/
categories:
  - Linux
tags:
  - openvpn
---
本文讲述使用 vps 搭建 [openvpn][1]，以及在 Gentoo 中使用 [chnroutes][2] 修改路由表，将国内 ip 的访问重新指定为非 vpn 线路。

## 条件

服务器为支持 Tun/Tap 的 vps: 我自己的 Directspace 的 vps，默认是没有开启的，需发 Tk 开启。免费的 AWS EC2 是默认开启了的。我这里的 vps 安装的操作系统为 Debian。

客户端为 Linux：我用的为 Gentoo，其他发行版的脚本和启动 openvpn 的方式可能会不一样

## 在 vps 中安装与配置 openvpn 服务器

安装 openvpn 软件，很简单：

{% highlight bash %}aptitude install openvpn{% endhighlight %}

生产服务器证书：

{% highlight bash %}
cd /etc/openvpn/
cp /usr/share/doc/openvpn/examples/easy-rsa/2.0/* .
source ./vars
./clean-all
./build-ca
./build-key-server server
./build-dh
./build-key client1
{% endhighlight %}

在 ./build-key-server server 和 ./build-key client1 两步时会询问是否签名，回答是，如下，其他使用默认即可：

{% highlight bash %}...
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y
{% endhighlight %}

好了，此时，在 /etc/openvpn/keys 目录下面就有了服务器和客户端所需的证书  
下面修改服务器配置文件：/etc/openvpn/server.conf，我只提取有用部分：

{% highlight bash %}port 53
proto udp     # 我这里采用 udp 的 53 端口通信，和 dns 解析用的端口一样 ...
dev tun
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/server.crt
key /etc/openvpn/keys/server.key
dh /etc/openvpn/keys/dh1024.pem    # 以上4个指定服务器证书的路径
server 10.8.0.0 255.255.255.0      # 配置ip地址段
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"    # 默认路由走 vpn，后面在客户端设置时会将国内的 ip 再修改回非 vpn 的路由
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"    # 使用 Google 的 DNS
client-to-client    # vpn 客户端之间可以通信
duplicate-cn        # 多个客户端可以使用同一个证书
keepalive 10 120
comp-lzo
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
log /var/log/openvpn.log
verb 3
{% endhighlight %}

一些注释都在上面了。然后启动 openvpn 服务器了

{% highlight bash %}/etc/init.d/openvpn start
{% endhighlight %}

服务器上面还有最后两件事情：打开路由转发选项和配置 SNAT 规则  
打开路由转发：修改 /etc/sysctl.conf 文件，将 net.ipv4.ip_forward=1 前面的注释去掉

{% highlight bash %}...
net.ipv4.ip_forward=1
...
{% endhighlight %}

然后执行命令：

{% highlight bash %}sysctl -p
{% endhighlight %}

最后设置 MASQUERADE 规则，将 10.8.0.0 网段过来的数据包转发出去后都将源地址修改为本机的 IP 地址，以使回复包能够正确的回到 vps 上。命令：

{% highlight bash %}iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
{% endhighlight %}

其中 eth0 要改为 vps 上面的网络接口名称，比如在 Directspace 上面就为 venet0；  
当然这样只是当前生效，vps 重启之后还得重新配置 MASQUERADE 规则，可以使用如下语句将当前的 iptables 规则保存起来，然后在开机的时候加载

{% highlight bash %}iptables-save > /etc/openvpn/iptables
{% endhighlight %}

在 /etc/rc.loacl 中加入如下语句：

{% highlight bash %}iptables-restore < /etc/openvpn/iptables
{% endhighlight %}

好了，服务器配置就完成了

## 在 Gentoo 下配置 openvpn 客户端

首先同样为安装 openvpn 软件，我这里是 Gentoo，你需要使用你用的发行版的包管理安装

{% highlight bash %}emerge openvpn openresolv iproute2
{% endhighlight %}

安装 openresolv 是为了需要 resolvconf 程序，用于修改 dns 配置的，其他发行版可能不太一样  
安装 iproute2 是为了获得 ip 程序，在修改路由时会使用到。  
客户端需要使用在配置服务器端时生成的 key，需要如下三个文件：

{% highlight bash %}ca.crt  client1.crt  client1.key
{% endhighlight %}

同样，我这里也放到了 /etc/openvpn/keys 目录  
然后配置 /etc/openvpn/client.conf

{% highlight bash %}client
dev tun
proto udp
remote xxx.xxx.xxx.xxx 53      # 这里需要换成你自己的服务器，域名或 IP
resolv-retry infinite
nobind
persist-key
persist-tun
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/client1.crt
key /etc/openvpn/keys/client1.key    # 你自己的证书路径
comp-lzo
verb 3
redirect-gateway
script-security 2
{% endhighlight %}

创建启动脚本

{% highlight bash %}ln -vs openvpn /etc/init.d/openvpn.client
{% endhighlight %}

然后使用如下命令启动即可

{% highlight bash %}/etc/init.d/openvpn.client start
{% endhighlight %}

但是，这样启动之后，所有的流量都回走 vpn，因此访问国内资源时很慢，且有可能会有限制，所以接下来就需要使用 chnroutes 的方法将访问国内 ip 的路由修改为非 vpn 的线路  
下载 [chnroutes.py][3] 文件，然后执行

{% highlight bash %}python chnroutes.py -p linux
{% endhighlight %}

之后，目录下面就会多出两个文件：

{% highlight bash %}ip-pre-up ip-down
{% endhighlight %}

然后将这两个文件拷贝到 /etc/openvpn/ 目录并改名，同时加上可执行权限：

{% highlight bash %}cp ip-pre-up /etc/openvpn/openvpn.client-up.sh
cp ip-down /etc/openvpn/openvpn.client-down.sh
chmod 755 /etc/openvpn/openvpn.client-up.sh
chmod 755 /etc/openvpn/openvpn.client-down.sh
{% endhighlight %}

Gentoo 下面会在 vpn 启动和停止的时候自动调用上面两个脚本。这时再运行

{% highlight bash %}/etc/init.d/openvpn.client start
{% endhighlight %}

过不一会儿，使用 ip 命令查看路由表信息就可以看到国内的 ip 都走了非 vpn 的线路

{% highlight bash %}$ ip r
0.0.0.0/1 via 10.8.0.9 dev tun0
default via 192.168.1.1 dev eth0  metric 2
1.0.1.0/24 via 192.168.1.1 dev eth0
1.0.2.0/23 via 192.168.1.1 dev eth0
1.0.8.0/21 via 192.168.1.1 dev eth0
1.0.32.0/19 via 192.168.1.1 dev eth0
1.1.0.0/24 via 192.168.1.1 dev eth0
1.1.2.0/23 via 192.168.1.1 dev eth0
1.1.4.0/22 via 192.168.1.1 dev eth0
1.1.8.0/21 via 192.168.1.1 dev eth0
...
{% endhighlight %}

好了，现在可以打开 Youtube 看下视频了，AWS EC2 的免费 vpn 速度可以到 400K，已经很不错了，不过流量每月只有 30G :)

 [1]: http://openvpn.net/ "openvpn"
 [2]: https://github.com/fivesheep/chnroutes "chnroutes"
 [3]: https://github.com/fivesheep/chnroutes/blob/master/chnroutes.py "chnroutes.py"
