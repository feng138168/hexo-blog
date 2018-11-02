---
title: 什么是CSRF
date: 2018-09-21 11:27:27
categories: 
- http
---
# 前言
之所以会想了解CSRF，是因为之前去腾讯面试，在谈到跨域问题时，被问到“了不了解CSRF”，当时没能会打上来，所以这次想做一个简单的了解，日后有这方面的接触，再进行详细的补充。
<!-- more -->
# 什么是CSRF
CSRF(Cross-site request forgery)，中文名称：跨域请求伪造。是一种利用cookie，伪造用户请求，对相关网站进行信息窃取或者修改的行为。
# CSRF基本原理
用户A访问网站B，并且对网站进行了登录授权。

在网站B没有关闭的情况下，另外开启tab栏，打开恶意网站C。

此时网站C用`<img src="网站B">`的方式，在用户不知情的情况下，由于原本的会话没有过期，带上了对网站B的授权cookie，访问了网站B，实现了对网站B的恶意修改信息的行为。
# 如何防止CSRF
## 检查Referer
http请求的referer可以获取到当前请求页面地址，服务器在接收到敏感访问时，可以先校验referer是否在正常域名下。

**目前有许多可以强行修改referer的方式，部分浏览器也因为referer可能会泄露用户的信息，所以强行关闭了referer，会导致服务端的误判，导致请求没办法正常进行，所以目前不建议这种方式。**
## 设置token
在获取到登录授权时，返回一串加密的token，前端将token保存在其他位置，在发送请求时带上token，服务端再对token进行检查

# 参考网站
【知行合一】http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html#!comments

【饥人谷_庄海鑫】https://www.jianshu.com/p/e825e67fcf28
