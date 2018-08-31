---
title: 使用axios时跨域失败的一些处理方法笔记
date: 2017-09-31 11:37:42
categories: 
- http
---
一开始需要前后端分离时经常面对跨域失败这个问题，出现问题的原因主要是因为服务端和客户端不在同一个域中，实现跨域的方法有几种，类似于JSONP，iframe，CORS资源共享几种方法，前面两种都有自身的局限性，只能使用get访问请求，所以使用比较多的通过CORS资源共享来实现跨域。
<!-- more -->

有关CORS资源共享的详细介绍，可以参考阮一峰的博客，里面有较为详细的介绍。因为之前都是使用vue的时候遇到的，所以通过axios为ajax的框架，nodejs为后台代码简单贴一下代码。
# 前端的设置

```
//url表示地址，data是一个json对象的字符串
//需要特别注意的是axios会自动验证data的类型，此处data是字符串，所以contentType为application/x-www-form-urlencoded
axios.post(url, data).then(res => {
  //成功的回调
}).catch(err => {
  //错误的回调
})
```
仅仅针对最简单的post请求，前端不需要设置什么
# 后端的设置
```
router.options('/upload', function* (){
  this.set('Access-Control-Allow-Methods', 'POST');//允许的请求方式为POST
  this.set('Access-Control-Allow-Origin', '*');//允许的请求链接为所有
  this.set('Access-Control-Allow-Headers', 'Content-type');//允许请求链接修改的请求头
});
```
根据以上设置，已经基本能实现跨域，下面就根据我所遇到的跨域失败的情况做一个简单总结

# 跨域失败问题排查
- 查看你的Request Method是否为OPTIONS

  当你的请求需要修改服务器资源时，类似于POST DELETE PUT请求，服务器为了安全，会先发送一个OPTIONS请求来确认是否安全

  当服务器没有允许OPTIONS请求访问，或不支持OPTIONS请求时，请求将无法进行

- 如何避免请求时，先发送OPTIONS

  不使用非GET和POST的请求

  使用常规的三个contentType进行访问，包括（application/x-www-form-urlencoded multipart/form-data text/plain）