---
title: Blog 换为 Jekyll
author: zlbruce
layout: post
permalink: /2015/07/08/use-jekyll/
categories:
  - Blog
tags:
  - Blog
  - Jekyll
  - 多说
---
由于受不了经常性的 OOM，所以现在把 Blog 换到了 [Jekyll][Jekyll] 上了，评论系统采用了[多说][duoshuo]。主题用的是 [Freshman21][Freshman21]，稍微做了下修改。

文章和评论应该都完全迁移过来了。不过现在还存在一个问题，就是多说的评论中第三方的头像会使用非 HTTPS 的连接，导致 Blog 的页面不完全是 HTTPS 的。

当前是没有太好的办法解决了，就先这样吧。

 [Jekyll]: http://jekyllrb.com/ 'Jekyll'
 [duoshuo]: http://duoshuo.com/ '多说'
 [Freshman21]: https://github.com/yulijia/freshman21 'Freshman21'
