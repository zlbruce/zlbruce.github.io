---
title: 在 Apache 2.4 中使用 mod_spdy
author: zlbruce
layout: post
permalink: /2014/10/23/install-mod_spdy-on-apache2-4/
categories:
  - Linux
tags:
  - apache2
  - spdy
---
新的 VPS 使用的操作系统 Ubuntu 14.04.1 LTS，其 Apache2 的版本号为 2.4.7-1ubuntu4.1，而 [Google 提供的 mod_spdy][1] 安装包依赖的 Apache2 的版本为 2.2，在安装 mod_spdy 的时候会出错，提示如下：

{% highlight bash %}
# dpkg -i mod-spdy-beta_current_amd64.deb 
Selecting previously unselected package mod-spdy-beta.
(Reading database ... 111842 files and directories currently installed.)
Preparing to unpack mod-spdy-beta_current_amd64.deb ...
Unpacking mod-spdy-beta (0.9.4.3-r420) ...
dpkg: dependency problems prevent configuration of mod-spdy-beta:
 mod-spdy-beta depends on apache2.2-common; however:
  Package apache2.2-common is not installed.

dpkg: error processing package mod-spdy-beta (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 mod-spdy-beta
{% endhighlight %}

经过一番 Google 之后，找到了这个[支持 Apache 2.4的 mod_spdy][2]，安装方法如下，  
首先下载源码进行编译：

{% highlight bash %}
$ sudo apt-get -y install git g++ apache2 libapr1-dev libaprutil1-dev patch binutils make devscripts
$ git clone https://github.com/eousphoros/mod-spdy.git
$ cd mod-spdy/src
$ git checkout apache-2.4.7
$ ./build_modssl_with_npn.sh
$ chmod +x ./build/gyp_chromium
$ make BUILDTYPE=Release
$ cp out/Release/libmod_spdy.so mod_spdy.so
{% endhighlight %}

这个时候会出来两个 Apache2 模块，mod\_ssl.so 和 mod\_spdy.so，把这两个文件拷贝到 Apache2 的模块目录：

{% highlight bash %}
$ sudo mv /usr/lib/apache2/modules/mod_ssl.so /usr/lib/apache2/modules/mod_ssl.so.bak
$ sudo cp mod_ssl.so /usr/lib/apache2/modules
$ sudo cp mod_spdy.so /usr/lib/apache2/modules
{% endhighlight %}

mod_ssl.so 会比原来的大不少，那是因为为了不影响系统的 openssl 库，它把 openssl 库给静态编译进去了。  
最后设置模块，让 Apache2 启动的时候加载 mod_spdy 模块就行了：

{% highlight bash %}
$ echo "LoadModule spdy_module /usr/lib/apache2/modules/mod_spdy.so" | sudo tee /etc/apache2/mods-available/spdy.load
$ echo "SpdyEnabled on" | sudo tee /etc/apache2/mods-available/spdy.conf
$ sudo a2enmod ssl
$ sudo a2enmod spdy
$ sudo service apache2 restart
{% endhighlight %}

PS：之前研究 IE11 的时候发现他用的 [ALPN 而不是 NPN][3]，花了一下午时间让 mod_spdy 支持了 ALPN 协议，结果回到家才发现 IE11 现在也支持了 NPN，记得以前是不支持 NPN 的呀，简直白折腾了。

 [1]: https://developers.google.com/speed/spdy/mod_spdy/ "mod_spdy"
 [2]: https://github.com/eousphoros/mod-spdy "eousphoros/mod-spdy"
 [3]: {{ site.url }}/2013/07/19/npn-and-alpn/ "NPN 与 ALPN"
