---
title: IE 不支持 deflate
author: zlbruce
layout: post
permalink: /2010/03/09/ie-do-not-support-deflate/
categories:
  - 笔记
tags:
  - deflate
  - gzip
  - IE
  - zlib
---
最近在做http的压缩，稍微了解了下gzip,deflate,zlib的关系：

  * [deflate(RFC1951)：][1]一种压缩算法，使用LZ77和哈弗曼进行编码；
  * [zlib(RFC1950)：][2]一种格式，是对deflate进行了简单的封装；
  * [gzip(RFC1952)：][3]一种格式，也是对deflate进行的封装。

可以看出deflate是最核心的算法，而zlib和gzip格式的区别仅仅是头部和尾部不一样，而实际的内容都是deflate编码的，即：  
gzip = gzip头 + deflate编码的实际内容 + gzip尾  
zlib = zlib头 + deflate编码的实际内容 + zlib尾

在HTTP/1.1的[RFC2616文档][4]中说明了Content-Encoding字段的值可以为：gzip, deflate等。

gzip格式大家都支持的很好很标准，这里说下deflate格式，[Content-Encoding的说明][5]中指出deflate指的是在RFC1950说明的zlib格式。也就是说当Content-Encoding为deflate时，内容应该为zlib格式。

但是，实际上，如果真的按照这个标准来，那么在IE上面是打不开页面的，包括IE6，IE7，IE8，提示为一片空白或者出错。但是在其他的浏览器如Firefox，Chrome，Opera等上面都能正常打开。要让IE能够正常打开页面，内容必须是deflate原始格式的数据，即去掉zlib头和zlib尾。不知道IE为什么不修改这个Bug，按理说在IE6就出现的这种很简单的问题，IE8不应该出现才对。

为了照顾IE，只好在压缩deflate的时候去掉zlib头和zlib尾，还好其他的浏览器也都能正常处理这种原始的deflate格式。  
当然，这样的话，那些受够了IE Only的人们倒是可以创建出一个IE不能正常访问的网站了。

 [1]: http://www.ietf.org/rfc/rfc1951.txt "deflate"
 [2]: http://www.ietf.org/rfc/rfc1950.txt "zlib"
 [3]: http://www.ietf.org/rfc/rfc1952.txt "gzip"
 [4]: http://www.w3.org/Protocols/rfc2616/rfc2616.html "HTTP 1.1"
 [5]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.5 "Content-Encoding"