---
title: 认识http/2
date: 2019-02-24 23:12:48
categories: http
tags: [http]
---

## 介绍
HTTP/2（超文本传输协议第2版，最初命名为HTTP 2.0），简称为h2（基于TLS/1.2或以上版本的加密连接）或h2c（非加密连接）

HTTP/2标准于2015年5月以RFC 7540正式发表。多数主流浏览器已经在2015年底支持了该协议。此外，根据W3Techs的数据，在2017年5月，在排名前一千万的网站中，有13.7%支持了HTTP/2。

## 协议之间的比较
### HTTP/2与HTTP/1.1比较
HTTP/2 相比 HTTP/1.1 的修改并不会破坏现有程序的工作，但是新的程序可以借由新特性得到更好的速度。

[HTTP/2: the Future of the Internet](https://http2.akamai.com/demo)是 Akamai 公司建立的一个官方的演示，用以说明 HTTP/2 相比于之前的 HTTP/1.1 在性能上的大幅度提升。 同时请求 379 张图片，从Load time 的对比可以看出 HTTP/2 在速度上的优势。
{% asset_img images/a8c500242cca3edb042b194d90763658_r.jpg %}

http/2和HTTP1.X相比的新特性:
1. **新的二进制格式（Binary Format）**，HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑http/2的协议解析决定采用二进制格式，实现方便且健壮。
2. **多路复用（MultiPlexing）**，即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
3. **header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，http/2使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
4. **服务端推送（server push）**，同SPDY一样，http/2也具有server push功能。

### HTTP/2与SPDY的比较
http/2可以说是SPDY的升级版（其实原本也是基于SPDY设计的），但是，http/2 跟 SPDY 仍有不同的地方，如下：
1. http/2 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS
2. http/2 消息头的压缩算法采用 HPACK，而非 SPDY 采用的 DEFLATE

HTTP/2的开发基于SPDY进行跃进式改进。在诸多修改中，最显著的改进在于，HTTP/2使用了一份经过定制的压缩算法，基于霍夫曼编码，以此替代了SPDY的动态流压缩算法，以避免对协议的Oracle攻击——这一类攻击以CRIME为代表。此外，HTTP/2禁用了诸多加密包，以保证基于TLS的连接的前向安全。

## HTTP/2的升级改造
1. http/2其实可以支持非HTTPS的，但是现在主流的浏览器像chrome，firefox表示还是只支持基于 TLS 部署的http/2协议，所以要想升级成http/2还是先升级HTTPS为好。
2. 当你的网站已经升级HTTPS之后，那么升级http/2就简单很多，如果你使用NGINX，只要在配置文件中启动相应的协议就可以了，可以参考[NGINX白皮书](https://www.nginx.com/wp-content/uploads/2015/09/NGINX_HTTP2_White_Paper_v4.pdf)，[NGINX配置http/2官方指南](https://www.nginx.com/blog/nginx-1-9-5/)。
3. 使用了http/2那么，原本的HTTP1.x怎么办，这个问题其实不用担心，http/2完全兼容HTTP1.x的语义，对于不支持http/2的浏览器，NGINX会自动向下兼容的。

## http/2的多路复用和HTTP1.X中的长连接复用的区别
- HTTP/1.* 一次请求-响应，建立一个连接，用完关闭；每一个请求都要建立一个连接；
- HTTP/1.1 Pipeling解决方式为，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；
- HTTP/2多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行；

具体如图：
{% asset_img images/138606-37ea7846b10ea092.png %}

## 服务器推送
服务端推送能把客户端所需要的资源伴随着index.html一起发送到客户端，省去了客户端重复请求的步骤。正因为没有发起请求，建立连接等操作，所以静态资源通过服务端推送的方式可以极大地提升速度。

具体如下：
- 普通的客户端请求过程：
{% asset_img images/138606-d6b704a63c41587c.png %}

- 服务端推送的过程：
{% asset_img images/138606-180318d7f001446f.png %}

## 头部压缩
假定一个页面有100个资源需要加载（这个数量对于今天的Web而言还是挺保守的）, 而每一次请求都有1kb的消息头（这同样也并不少见，因为Cookie和引用等东西的存在）, 则至少需要多消耗100kb来获取这些消息头。http/2可以维护一个字典，差量更新HTTP头部，大大降低因头部传输产生的流量。
具体参考：[HTTP/2 头部压缩技术介绍](https://imququ.com/post/header-compression-in-http2.html)

## 参考文章
[HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2#%E5%8D%8F%E8%AE%AE%E7%9A%84%E5%88%B6%E5%AE%9A)
[HTTP1.0、HTTP1.1和http/2的区别](https://www.jianshu.com/p/be29d679cbff)
[HTTP/2.0 相比1.0有哪些重大改进？](https://www.zhihu.com/question/34074946)

