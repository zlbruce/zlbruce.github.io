---
title: Pagespeed 笔记
author: zlbruce
layout: post
permalink: /2012/12/27/pagespeed-note/
categories:
  - 笔记
tags:
  - http
  - pagespeed
---
这是我这几天看 pagespeed 的一些笔记，由于我并不是搞 web 开发的，文中的内容只是我自己的一些理解，所以可能并不一定正确。

[pagespeed][1] 是 google 出品的一款 apache 模块，其中有多达 40 种的优化手段，可以帮助提升网站的加载速度。其主要优化方法有以下几种：

  * 减少页面的大小
  * 减少浏览器的连接数量
  * 提高浏览器渲染速度

## 减少页面的大小

在减少页面大小方面，pagespeed 提供了以下选项：

  * Collapse Whitespace
  * Combine Heads
  * Elide Attributes
  * Minify JavaScript
  * Optimize Image
  * Remove Comments
  * Remove Quotes
  * Rewrite CSS
  * Rewrite Style Attributes
  * Trim URLS

这些选项所做的基本上就是以下几点：

  * 对于 html, css, js 文件： 
      * 去掉不必要的空格
      * 去掉注释
  * 单独对于 js 文件： 
      * 类似 [jsmin][2] 方法压缩 js
  * 对于图片： 
      * 将小图片进行内联，目的是为了减少连接数
      * 重新压缩图片，以减少大小
      * png 转换为 jpg，jpg 转换为 webp

这每一个优化选项在开启了 gzip 压缩的服务器上面效果可能都不是很明显，不过一点一点的积累之后效果也是可以的。对于那些没有对 js 压缩过的网页，可能 Minify JavaScript 效果是最好的。

## 减少浏览器的连接数量

一般来说，在网页方面，减少连接数要比减少文件大小效果要好。

因为在有 gzip 压缩的情况下面，页面都是比较小的，而且能减少的大小也是很小的。有时候减少文件大小并不能减少传输的包个数，比如：4K的文件优化后为 3K，其实都需要 3 个 TCP 包才能传输完成。

而一个请求的代价则会多很多，首先，请求头部与应答头部的开销；其次，多一个请求可能会多一个 TCP 连接，新建 TCP 连接的开销。

在减少浏览器连接数上，pagespeed 提供了以下选项：

  * Combine CSS
  * Flatten CSS Imports
  * Inline CSS
  * Combine JavaScript
  * Inline JavaScript
  * Move CSS Above Scripts
  * Configuration file directive to shard domains
  * Sprite Images

看名字就能猜到其功能。

对于 css：主要就是将页面中的多个 css 合并为一个，将小的 css 文件内联到页面中，将 imports 的 css 也进行内联。

对于 js：主要也是将多个 js 合并为一个，将小的 js 文件进行内联。

对于 css 中的背景图片，Sprite Images 可以将多个背景图片合并为一个。

## 提高浏览器渲染速度

提高浏览器渲染速度其实是比较难的，不过 Google 自己就是做浏览器的，清楚的知道什么样的网页可以更快的进行渲染。可以使用的选项如下：

  * Convert Meta Tags
  * Defer JavaScript
  * Inline Preview Images
  * Lazyload Images
  * Move CSS to Head
  * Optimize Image
  * Rewrite Style Attributes

感觉比较有效果的是 Inline Preview Images 和 Lazyload Images。

Inline Preview Images 打开之后先加载低画质的图片，待高画质图片下载完成之后在加载高画质图片。

Lazyload Images 则只加载显示区域的图片。淘宝的页面应该就是使用类似这个的技术。

## 缓存改进

其实这个可以归为减少浏览器连接数，不过我还是把他单独列出来了，主要思想就是把资源尽可能的缓存在浏览器，这样浏览器访问页面的时候就直接使用本地缓存而不用向服务器发送请求了。提供的选项如下：

  * Extend Cache
  * Extend Cache PDF
  * Local Storage Cache
  * Outline CSS
  * Outline Javascripts

Extend Cache 修改资源的名称，在原有名称后面加上该资源的 hash 值，然后在资源的 http 应答头部中将缓存时间设置很长（比如 1 年），那么后续的页面如果还有引用该资源，则不会向服务器进行请求。如果资源有改动，那么 hash 值就会改变，因此浏览器会认为是不同的资源，然后发送请求获取新资源，当然引用资源的 html 页面是不能被缓存的。

Outline CSS/Javascripts 将页面中内联的 CSS/JS 进行外联，这样的好处是：CSS/JS 可以被缓存，如果多个页面用到同一个资源的话，可以节省传输量；如果 html 经常改动，那么 CSS/JS 的缓存也能减少传输量；如果 CSS/JS 很长，外联可以使 html 文件变小，可以让浏览器先加载 html 进行显示。当然外联的坏处也是显然的：增加了浏览器的请求数。

 [1]: https://developers.google.com/speed/pagespeed/mod "pagespeed"
 [2]: http://www.crockford.com/javascript/jsmin.html "jsmin"