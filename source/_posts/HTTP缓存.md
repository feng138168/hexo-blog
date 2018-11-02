---
title: HTTP缓存
date: 2018-10-31 19:43:26
categories: 
- http
---
# 前言
一直以来都对浏览器缓存很感兴趣，但总是一知半解，只是很碎片化的了解一些相关知识，这次就打算把所有的碎片整合一下，基于我目前的理解整理一份总结。等以后认识的更全面了，再逐步添加修改。
<!-- more -->
# 浏览器缓存的作用
合理使用浏览器的缓存，可以有效的减少请求的访问次数，一个是减小服务器压力，更重要的是能够降低页面的加载时间。
# HTTP缓存的几种形式及实现
## Pragma
这是一种比较古老的请求头，只建议在向下兼容http1.0的时候使用，当值为`Pragma:no-cache`，效果和`Cache-Control:no-cache`一样。
```
res.setHeader('Pragma', 'no-cache');
```
以nodejs为例，文件每次都会向服务器重新发起请求，哪怕设置了其他缓存方式，具体优先级待会会详细介绍。
## Expires
`Exprires`也是兼容http1.0的一项请求头。

通过服务端设置，给`Response Headers`添加一个GMT（格林尼治时间）作为`Express`的响应头，客户端在发请求前，判断`Express`的时间是否已经过期来决定是否向服务端发送请求。
```
// 过期时间在当前时间下添加5秒
const expires = new Date(Date.parse(new Date())+5000);
res.setHeader("Expires", expires.toUTCString());
```
![](https://res.bookingtee.com/opc/base/81bf4ca49ef31c2f.png)

其中`Date`为上一次请求的时间

由于客户端在做判断的时候取的是自己电脑的时间，所以当用户的电脑时间出现问题时，会影响到缓存的正常使用。
## Cache-Control
当http发展到1.1时，简单的请求头已经无法覆盖所有的操作，所以需要更加灵活的`Cache-Control`来充实浏览器缓存的形式。

客户端可以在HTTP请求中使用的标准`Cache-Control`
```
//告诉服务端，愿意接收一个请求时间为seconds秒的资源
Cache-Control: max-age=<seconds>
//告诉服务端，愿意接收一个超出缓存时间seconds秒的资源，如果未定义，则允许超出任意时间
Cache-Control: max-stale[=<seconds>]
//告诉服务端，希望接收一个seconds秒内被更新过的资源
Cache-Control: min-fresh=<seconds>
//告诉服务端，不直接使用缓存，跳过强缓存的步骤
Cache-control: no-cache
//告诉服务端，所有内容都不被缓存到浏览器中
Cache-control: no-store
//告诉服务端，希望获取实体数据没有被转换过（如压缩）的资源
Cache-control: no-transform
//告诉服务端，希望获取缓存的内容（若有）
Cache-control: only-if-cached
```
服务器可以在响应中使用的标准`Cache-Control`
```
//缓存必须在使用之前验证旧资源的状态，并且不可使用过期资源
Cache-control: must-revalidate
//不直接使用缓存，跳过强缓存的步骤
Cache-control: no-cache
//所有内容都不被缓存到浏览器中
Cache-control: no-store
//不得对资源进行转换或转变。Content-Encoding, Content-Range, Content-Type等HTTP头不能由代理修改。例如，非透明代理可以对图像格式进行转换，以便节省缓存空间或者减少缓慢链路上的流量
Cache-control: no-transform
//表明响应可以被任何对象（包括：发送请求的客户端，代理服务器，等等）缓存
Cache-control: public
//表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）,可以缓存响应内容
Cache-control: private
//与must-revalidate作用相同，但它仅适用于共享缓存（例如代理），并被私有缓存忽略
Cache-control: proxy-revalidate
//设置缓存存储的最大周期，超过这个时间缓存被认为过期(单位秒)。与Expires相反，时间是相对于请求的时间
Cache-Control: max-age=<seconds>
//覆盖max-age 或者 Expires 头，但是仅适用于共享缓存(比如各个代理)，并且私有缓存中它被忽略
Cache-control: s-maxage=<seconds>
```
扩展Cache-Control指令(不是核心HTTP缓存标准文档的一部分，有兼容问题)
```
//表示响应正文不会随时间而改变。资源（如果未过期）在服务器上不发生改变，因此客户端不应发送重新验证请求头（例如If-None-Match或If-Modified-Since）来检查更新，即使用户显式地刷新页面
Cache-control: immutable 
//表明客户端愿意接受陈旧的响应，同时在后台异步检查新的响应。秒值指示客户愿意接受陈旧响应的时间长度
Cache-control: stale-while-revalidate=<seconds>
//表示如果新的检查失败，则客户愿意接受陈旧的响应。秒数值表示客户在初始到期后愿意接受陈旧响应的时间
Cache-control: stale-if-error=<seconds>
```
下面是nodejs通过给`Cache-Control`添加了过期时间一个小时，每次加载文件时，客户端会对Date进行比对，知道过期，否则不会向服务端发送新的请求
```
res.setHeader('Cache-Control', 'max-age=3600');
```
![](https://res.bookingtee.com/opc/base/d5767a41a577229c.png)

`Cache-Control`的比对比较复杂，是需要响应头和请求头同时作用的

（这里关于响应头和请求头之间互相冲突时，客户端是如何决定缓存结果的，我在后面再找时间添加）
## Last-Modified
第一次请求资源时，会将资源最后更改的时间以`Last-Modified: GMT`的形式加在实体首部上一起返回给客户端，当再一次发起请求时，客户端会带上返回的GMT，赋值给`If-Modified-Since`，若服务端判断时间没有更改，则返回`304 NOT MODIFIED`
```
const stats = fs.statSync(pathname);
const lastModified = stats.mtime.toUTCString();
const ifModifiedSince = "If-Modified-Since".toLowerCase();
res.setHeader("Last-Modified", lastModified);
//校验时判断下当前文件的更新时间和请求返回的时间是否相同
if (
  req.headers[ifModifiedSince] &&
  lastModified == req.headers[ifModifiedSince]
) {
  //返回
  res.writeHead(304, "Not Modified");
}
```
![](https://res.bookingtee.com/opc/base/52d7f5f0c5abc355.png)

服务端比对服务器中的文件和请求头中的更新时间，发现时间一致，返回304，客户端不再重新更新文件。

PS：一般这时候不会马上出现304的情况，会发现浏览器根本没有发送请求，而是直接拿了本地的缓存。

![](https://res.bookingtee.com/opc/base/29b7c1ce94653dce.png)

（此处内容是腾讯Bugly分享截取）这是因为浏览器的一种缓存过期策略。在没有提供任何浏览器缓存过期策略的情况下，浏览器遵循一个启发式缓存过期策略：
>根据响应头中2个时间字段 Date 和 Last-Modified 之间的时间差值，取其值的10%作为缓存时间周期。

以下完整的缓存策略三要素：

![](https://pic4.zhimg.com/v2-1494676f664d40eff41e37bfbc522323_r.jpg)

这里需要注意的点，根据图中的公式可以看出，如果上次缓存的文件时间较长，加上仅仅配置了`Last-Modified`，有可能导致用户一直无法拿到最新的文件。
## ETag
服务端在第一次请求资源时，会在响应头`ETag`上添加一个由某种算符得出的随机字符串，当客户端再一次请求该项资源时，会将之前缓存的字符串，自动带上给`If-None-Match`，服务端将得到的字符和服务器中的进行对比。如果不同，则重新抓取文件，若相同，则返回`304`。
```
//文件更新时重新设置etag(模拟服务器获取etag)
const etag = '123456789abcdef';
const ifNoneMatch = "If-None-Match".toLowerCase();
res.setHeader("Etag", etag);
if (
  req.headers[ifNoneMatch] &&
  etag == req.headers[ifNoneMatch]
) {
  res.writeHead(304, "Not Modified");
}
```
![](https://res.bookingtee.com/opc/base/49327d5d4bba86a7.png)
# 不同缓存方式的优先级关系
我这里就不写详细例子，直接将他们的优先级展示出来。

其中`Pragma>Cache-Control>Exprise`，优先级从左到右。

![](https://res.bookingtee.com/opc/base/1bf24d485898eed5.jpg)
## 强缓存
服务器返回`200 from cache`属于强缓存，客户端通过`Pragma、Cache-Control和Exprise`判断是否有缓存，如果有缓存，直接访问本地的缓存，不访问服务器，能更有效的节省资源。
### 磁盘缓存(from cache || from disk cache)
### 内存缓存(from memory cache)
## 协商缓存（弱缓存）
服务器返回`304 Not Modified`属于协商缓存，客户端通过向服务端发请求验证资源是否过期，若发现已经过期，则重新拉去，若没有过期，服务端返回304，客户端则从本地缓存中获取资源，不重新拉去。
# 分析不同缓存方式的优劣性
头部字段 | 优势 | 劣势
----|------|----
Pragma | 基于http1.0，兼容性强 | 可操作性不强，基本已经不再使用
Expires | 基于http1.0，兼容性比较强，客户端判断是否过期，无需向服务端确认  | 1.由于是基于用户本地时间进行计算，所以可能存在不准确的问题。2.强缓存在到期之前用户无法知道资源是否过期。
Cache-Control | 操作比较丰富，组合性强  | 同样由于是强缓存，可能存在更新不及时的问题
Last-Modified | 兼容性强，更新及时 | 资源的任何变化都会改变更新时间，可能会重复加载完全一样的文件
ETag | 文件的更新由服务端控制，操作性强，且兼容性强 | 计算ETag需要消耗性能，且不同的服务端计算的算法可能不一致，导致重复加载