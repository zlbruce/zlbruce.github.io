---
title: NPN 与 ALPN
author: zlbruce
layout: post
permalink: /2013/07/19/npn-and-alpn/
categories:
  - 笔记
tags:
  - spdy
---
现在 IE11 已经支持 SPYD 协议了，不过在我测试后发现他和 Fx, Chrome 等其他浏览器所使用 TLS 扩展协议不一样。  
在 [SPDY 简介][1]中说到过，由于 SPDY 并没有一个确定的端口号，因此现有的 SPDY 协议都是跑在 HTTPS 的 443 端口上的，但是客户端和服务器端在 SSL 协商完成之后，服务器并不知道你是要使用 HTTP 还是 SPDY 协议，因此需要在进行 SSL 协商的同时，将上层的应用层协议给协商出来。  
以前 Fx 和 Chrome 是通过 NPN 扩展来进行的协商，而 IE11 则是使用的其他的协议，使用 wireshark 抓包后看到的扩展为“Extension: Unknown 16”，表示认不出来&#8230;&#8230;在 Google 了一番之后，终于找到了其协议的文档，同时也知道了该扩展的名字：ALPN

## NPN

NPN 的全称为 Next Protocol Negotiation，其 RFC 草案已经过期了，基本的协商流程如下：

    Client                                      Server

    ClientHello (NPN extension)  -------->
                                                ServerHello (NPN extension &#038; list of protocols)
                                                Certificate*
                                                ServerKeyExchange*
                                                CertificateRequest*
                                 <--------      ServerHelloDone
    Certificate*
    ClientKeyExchange
    CertificateVerify*
    [ChangeCipherSpec]
    EncryptedExtensions
    Finished                     -------->
                                                [ChangeCipherSpec]
                                 <--------      Finished
    Application Data             <------->      Application Data


流程描述：

  1. 客户端通过 ClinetHello 发送一个空的 NPN 扩展字段
  2. 服务端通过 NPN 扩展返回支持的协议列表
  3. 客户端在 ChangeCipherSpec 之后 Finished 之前发送 EncryptedExtensions 选择某一个协议

## ALPN

ALPN 的全称为 Application Layer Protocol Negotiation，做为 NPN 的替代者，其基本的协商流程如下：

    Client                                                          Server

    ClientHello (ALPN extension &#038; list of protocols)  -------->
                                                                     ServerHello (ALPN extension &#038; selected protocols)
                                                                     Certificate*
                                                                     ServerKeyExchange*
                                                                     CertificateRequest*
                                                      <--------      ServerHelloDone
    Certificate*
    ClientKeyExchange
    CertificateVerify*
    [ChangeCipherSpec]
    Finished                                          -------->
                                                                     [ChangeCipherSpec]
                                                      <--------      Finished
    Application Data                                  <------->      Application Data


流程描述：

  1. 客户端通过 ALPN 扩展将自己支持的应用层协议发送给服务端
  2. 服务端选择其中某个协议，并将结果通过 ALPN 扩展发送给客户端
  3. SSL 协商完成后进行正常通讯

## 两者的区别

  * NPN 是服务器发送协议列表，由客户端进行选择。而 ALPN 则是客户端发送列表，由服务端选择
  * 在 NPN 中，最终的选择结果是在 ChangeCipherSpec 之后发送给服务端的，也就是说是被加密了的。而在 ALPN 中，所有的协商都是明文的
  * 在 NPN 中，为了防止服务器支持的协议泄露，客户端可以选择一个不在服务器端给出的列表中的协议。而 ALPN 中，服务器端只能选择在客户端发送的列表中的某个协议。

基本的区别就是以上几点，虽然看起来两者差别不大，不过似乎现在 NPN 协议已经被弃用，以后 Chrome 和 Google 的服务器都将支持 ALPN，而慢慢将 NPN 给淘汰掉。

## 参考链接

  * [draft-npn][2]
  * [draft-alpn][3]
  * [NPN and ALPN][4]

 [1]: {% post_url 2013-01-07-spdy-intro %} "SPDY 简介"
 [2]: http://tools.ietf.org/html/draft-agl-tls-nextprotoneg-04 "Transport Layer Security (TLS) Next Protocol Negotiation Extension"
 [3]: http://tools.ietf.org/html/draft-ietf-tls-applayerprotoneg-01 "Transport Layer Security (TLS) Application Layer Protocol Negotiation Extension"
 [4]: https://www.imperialviolet.org/2013/03/20/alpn.html "NPN and ALPN"
