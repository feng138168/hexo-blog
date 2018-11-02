---
title: promise
date: 2018-10-16 15:47:52
categories: 
- ES6
---
# 前言
之前在腾讯面试时曾被问到过这个，所以趁现在做一个技术总结。
<!-- more -->
# promise是什么
promise是ES6中的一个新的对象，用来处理异步操作。
# promise好在哪里
常见的情况如定时器和ajax，这些都是需要异步执行，所以我们只能通过回调函数去实现：
```
function callback() {
    console.log('Done');
}
console.log('before setTimeout()');
setTimeout(callback, 1000); // 1秒钟后调用callback函数
console.log('after setTimeout()');
```
这是一种比较常见的写法，但缺点也是很明显，如果需要嵌套的函数过多，函数嵌套层级过高，就会导致代码结构混乱，不方便阅读，也不利于代码的复用。

使用promise就能比较好的解决这个问题：
```
function test(resolve, reject) {
    var timeOut = Math.random() * 2;
    setTimeout(function () {
        if (timeOut < 1) {
            resolve('200 OK');
        }
        else {
            reject('timeout in ' + timeOut + ' seconds.');
        }
    }, timeOut * 1000);
}
var p1 = new Promise(test);
var p2 = p1.then(function (result) {
    console.log('成功：' + result);
});
var p3 = p2.catch(function (reason) {
    console.log('失败：' + reason);
});
/**
  简写写法
  new Promise(test).then(function(result){
    console.log('成功：' + result);
  }).catch(function (reason) {
    console.log('失败：' + reason);
  })
 */
```
函数test有两个形参，一个成功的回调resolve，一个失败的回调reject，当执行resolve时，会执行then中的回调函数，执行resolve时，会执行catch中的回调函数。

使用promise能降低代码的复杂度，用链式操作的方式也容易理解代码执行的顺利，方便代码阅读和维护。
# promise如何使用
已经在上面提到一种promise的基本写法，是单个回调的情况，还有一种比较常见的是多个回调函数串联在一起的情况。
```
// 0.5秒后返回input*input的计算结果:
function multiply(input) {
    return new Promise(function (resolve, reject) {
        console.log('calculating ' + input + ' x ' + input + '...');
        setTimeout(resolve, 500, input * input);
    });
}

// 0.5秒后返回input+input的计算结果:
function add(input) {
    return new Promise(function (resolve, reject) {
        console.log('calculating ' + input + ' + ' + input + '...');
        setTimeout(resolve, 500, input + input);
    });
}

var p = new Promise(function (resolve, reject) {
    console.log('start new Promise...');
    resolve(123);
});

p.then(multiply)
 .then(add)
 .then(multiply)
 .then(add)
 .then(function (result) {
    console.log('Got value: ' + result);
});

```
执行结果
```
start new Promise...

calculating 123 x 123...

calculating 15129 + 15129...

calculating 30258 x 30258...

calculating 915546564 + 915546564...

Got value: 1831093128
```
多个promise串联在一起，代码执行顺序清晰明了。

promise的链式操作是按照顺序执行的，所以不能随意改变顺序。
# 如何用es5实现promise
