---
title: 使用 dnsmasq 配合 openvpn
author: zlbruce
layout: post
permalink: /2011/07/20/dnsmasq-and-openvpn/
categories:
  - Linux
tags:
  - dnsmasq
  - openvpn
---
[使用 openvpn 并配合 chnroutes 脚本上网][1]已经很完美了，在访问国外的 ip 时自动使用 vpn 线路，在访问国内的 ip 时则自动选用非 vpn 线路，为什么还要使用 [dnsmasq][2] 呢？原因如下：

由于有域名污染，所以 dns 只能使用 8.8.8.8 或者 [opendns][3] 之类的进行解析；而使用国外的 dns 之后，在解析某些使用了智能 dns 的服务时，智能 dns 服务器往往会选择错 ip 地址，就会导致：

  1. 明明是国内的服务，却解析到一个国外的 ip，并使用 vpn 进行访问，导致速度慢
  2. 自己的电信的线路，却解析到网通的 ip，访问同样也会受到影响

为了解决上述问题，就需要将这些域名转发到自己所在的 isp 提供的 dns 进行解析。一般的 dns 服务器软件都提供这样的功能，不过这里用不到那么复杂的功能。而之前介绍 dns 缓存的时候使用到的 dnsmasq 也能提供转发功能，因此这里就继续使用 dnsmasq。配置极其简单，只需要在配置文件 /etc/dnsmasq.conf 中对 server 字段进行配置即可，比如我的配置:

    no-resolv
    no-poll
    server=8.8.8.8
    server=8.8.4.4
    server=/qq.com/192.168.1.1

表示默认使用 8.8.8.8 和 8.8.4.4 解析，当解析 *.qq.com 的时候使用 192.168.1.1 解析。你需要把 192.168.1.1 换成你 isp 提供的 dns 服务器。  
最后，启动 dnsmasq 服务，把自己的 dns 服务器指向 127.0.0.1 即可，由于我使用了 resolvconf，所以直接在 /etc/resolvconf.conf 中加入一下配置即可：

    name_servers=127.0.0.1

下面可以使用 dig 命令进行一个简单的测试：

{% highlight bash %}
$ dig @8.8.8.8 web2.qq.com
 
; <<>> DiG 9.8.0 <<>> @8.8.8.8 web2.qq.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6525
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0
 
;; QUESTION SECTION:
;web2.qq.com.			IN	A
 
;; ANSWER SECTION:
web2.qq.com.		221	IN	A	112.90.138.163
web2.qq.com.		221	IN	A	112.90.138.164
web2.qq.com.		221	IN	A	112.90.141.88
 
;; Query time: 217 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed Jul 20 00:17:59 2011
;; MSG SIZE  rcvd: 77
{% endhighlight %}
{% highlight bash %}
$ dig @192.168.1.1 web2.qq.com
 
; <<>> DiG 9.8.0 <<>> @192.168.1.1 web2.qq.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13332
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0
 
;; QUESTION SECTION:
;web2.qq.com.			IN	A
 
;; ANSWER SECTION:
web2.qq.com.		101	IN	A	183.62.126.217
web2.qq.com.		101	IN	A	183.60.3.126
web2.qq.com.		101	IN	A	121.14.74.112
web2.qq.com.		101	IN	A	183.60.3.84
 
;; Query time: 10 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Wed Jul 20 00:18:06 2011
;; MSG SIZE  rcvd: 93
{% endhighlight %}
{% highlight bash %}
$ dig @127.0.0.1 web2.qq.com
 
; <<>> DiG 9.8.0 <<>> @127.0.0.1 web2.qq.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23246
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0
 
;; QUESTION SECTION:
;web2.qq.com.			IN	A
 
;; ANSWER SECTION:
web2.qq.com.		94	IN	A	183.60.3.84
web2.qq.com.		94	IN	A	121.14.74.112
web2.qq.com.		94	IN	A	183.60.3.126
web2.qq.com.		94	IN	A	183.62.126.217
 
;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Jul 20 00:18:13 2011
;; MSG SIZE  rcvd: 93
{% endhighlight %}
{% highlight bash %}
$ dig @127.0.0.1 twitter.com
 
; <<>> DiG 9.8.0 <<>> @127.0.0.1 twitter.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57641
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0
 
;; QUESTION SECTION:
;twitter.com.			IN	A
 
;; ANSWER SECTION:
twitter.com.		19	IN	A	199.59.149.198
twitter.com.		19	IN	A	199.59.149.230
twitter.com.		19	IN	A	199.59.148.10
 
;; Query time: 205 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Jul 20 00:26:57 2011
;; MSG SIZE  rcvd: 77
{% endhighlight %}

可以看到使用 8.8.8.8 时解析到了网通的地址，而使用 127.0.0.1 后，则可以正确的解析为电信的地址。同时解析 twitter.com 也能正确的返回结果。

 [1]: https://zlb.me/2011/07/08/vps-openvpn/ "使用 vps 搭建 openvpn"
 [2]: http://thekelleys.org.uk/dnsmasq/doc.html "dnsmasq"
 [3]: http://www.opendns.com/ "opendns"
