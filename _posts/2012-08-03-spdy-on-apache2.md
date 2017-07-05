---
title: blog启用spdy
author: zlbruce
layout: post
permalink: /2012/08/03/spdy-on-apache2/
categories:
  - Blog
tags:
  - apache2
  - http
  - spdy
---
[spdy][1]的中文简介可以参看[维基百科的条目][2]。

如果你的服务器使用的是apache2.2或以上，且开启了mod_ssl的话，就可以通过安装[mod-spdy][3]模块来使用spdy。

如果没有开启ssl当前是无法使用spdy的，这是因为，当你在浏览器敲入一个网址的时候，浏览器并不知道服务器是否支持spdy，所以只能使用http协议。当使用ssl后，就可以通过NPN扩展知道上层的应用协议是http还是spdy了。估计要等spdy正式成为http 2.0标准之后才能在非ssl情况下使用吧。

需要注意的是，当我装完后发现，偶尔有些页面打开不完全，日志提示如下

    [Wed Aug 01 09:08:25 2012] [error] [client 183.12.64.115] PHP Fatal error: Cannot redeclare __() (previously declared in /var/www/blog/wp-admin/load-styles.php:17) in /var/www/blog/wp-admin/load-scripts.php on line 17, referer: https://zlb.me/wp-admin/update-core.php
    [Wed Aug 01 09:08:25 2012] [error] [client 183.12.64.115] PHP Parse error: syntax error, unexpected T_FUNCTION, expecting ')' in Unknown on line 0, referer: https://zlb.me/wp-admin/update-core.php
    [Wed Aug 01 09:08:37 2012] [error] [client 183.12.64.115] PHP Fatal error: Cannot redeclare __() (previously declared in /var/www/blog/wp-admin/load-styles.php:17) in /var/www/blog/wp-admin/load-scripts.php on line 17, referer: https://zlb.me/wp-admin/update-core.php
    [Wed Aug 01 09:14:27 2012] [error] [client 183.12.64.118] PHP Fatal error: Cannot redeclare __() (previously declared in /var/www/blog/wp-admin/load-styles.php:17) in /var/www/blog/wp-admin/load-scripts.php on line 17, referer: https://zlb.me/wp-admin/update-core.php

最后发现mod-spdy专门有个[Using mod_spdy with PHP][4]的页面。原来mod\_spdy使用了多线程，而mod\_php不是线程安全的，这样就会导致问题。解决方法就是使用mod\_fcgi替代mod\_php。

如果你是使用最新的Firefox或者Chrome浏览器，在访问我的Blog时，就已经是在使用spdy协议传输数据了，当然你访问的要是[https://zlb.me/][5]这个网址。

Chrome用户可以打开[chrome://net-internals/#spdy](chrome://net-internals/#spdy)页面来查看spdy的信息，而Firefox用户也可以安装[SPDY indicator扩展][6]来查看当前页面是否使用了spdy协议。

更新：在 Ubuntu 14.04 (Apache 2.4) 上安装 mod_spdy 可以参见我的另一篇文章：[在 Apache 2.4 中使用 mod_spdy][7]

 [1]: http://dev.chromium.org/spdy "spdy"
 [2]: http://zh.wikipedia.org/wiki/SPDY "spdy"
 [3]: https://code.google.com/p/mod-spdy/ "mod-spdy"
 [4]: https://developers.google.com/speed/spdy/mod_spdy/php "Using mod_spdy with PHP"
 [5]: https://zlb.me/ "https://zlb.me/"
 [6]: https://addons.mozilla.org/en-US/firefox/addon/spdy-indicator/ "SPDY indicator"
 [7]: https://zlb.me/2014/10/23/install-mod_spdy-on-apache2-4/ "在 Apache 2.4 中使用 mod_spdy"
