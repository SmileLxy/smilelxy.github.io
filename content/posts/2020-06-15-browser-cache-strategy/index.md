---
title: 浏览器缓存策略
author: xiaoyang.li
date: 2019-06-15
hero: ./images/poster.jpg
excerpt: 浏览器缓存策略
---

#### 缓存位置

- Service Worker
- Memory Cache
- Disk Cache
- Push Cache

##### 1. Service Worker

`Service Worker`的缓存与浏览器其他内建的缓存机制不同，它可以让我们自由控制缓存哪些文件、如何匹配缓存、如何读取缓存，并且缓存是持续性的。

##### 2. Memory Cache

`Memory Cache`即内存中的缓存，读取速度比较高效，但存储时效较短。一旦关闭页面，内存中的缓存就被释放。

内存缓存并不关心返回资源的 HTTP 缓存头`Cache-Control`是什么值，同时资源的匹配也并非仅仅是对 URL 做匹配。

##### 3. Disk Cache

`Disk Cache`即硬盘中的缓存，读取速度较慢，相对于`Memory Cache`胜在容量和存储时效性上。

##### 4. Push Cache

`Push Cache`（推送缓存）是 HTTP/2 中的内容，只会在会话中存在，一旦会话结束就被释放，缓存时间也较为短暂。

#### 缓存策略分析

##### 强缓存

不会向服务器发起请求，直接从缓存中读取资源，在 Chrome 的开发者工具中可发现`from disk cache`或`from memory cache`。

###### 1. Expires

缓存过期时间，用来指定资源的到期时间，是服务器端的具体时间点。 是 HTTP1 的产物，受限于本地时间。

###### 2. Cache-Control

用于控制网页缓存，在请求头或响应头中设置规则：

- public：表示响应可以被客户端和代理服务器缓存
- private：表示响应只可以被客户端缓存，默认值
- max-age=30：缓存 30 秒后过期
- s-maxage=30：覆盖 max-age，只在代理服务器生效
- no-store：不缓存任何响应
- no-cache：客户端缓存内容，但是是否使用需经过协商缓存来决定

###### 3. 对比

`Expires`是 HTTP 1.0 的产物，`Cache-Control`是 HTTP 1.1 的产物。同时存在的情况下，`Cache-Control`优先级较高。

##### 协商缓存

协商缓存即强制缓存失效后，浏览器携带缓存标识向服务器端发起请求，由服务器决定是否使用缓存。

###### 1. Last-Modified 与 If-Modified-Since

初次访问资源时，服务器端会在`response Header`中添加`Last-Modified`的 Header 头，值为该文件的最后修改时间。

再次访问时，浏览器会判断并添加`If-Modified-Since`的 Header 用于与服务器端作对比。

弊端在于：

1. 打开文件，即会对文件的`Last-Modifed`做出修改，即使文件未发生改变。
2. 只能以秒计时，当修改文件不可感知时，不够精确

###### 2. ETag 与 If-None-Match

`ETag`是服务器响应请求时，返回当前资源文件的唯一标识，只要资源发生变化，ETag 就会重新生成。 当再次请求时会携带`If-None-Match`的 Header。

###### 3. 区别

- 精确度上，`ETag`优于`Last-Modified`
- 性能上，`ETag`逊于`Last-Modified`，因为前者需要通过算法计算得到值
- 优先级上，`ETag`优先考虑。
