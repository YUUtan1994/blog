---
title: async await初步理解
date: 2019/01/21
comments: true

categories:
- 学习笔记
tags:
- ES2015
- async-await
thumbnail: https://cdn-images-1.medium.com/max/2400/1*2Nco5zYP_Xv-5-FgL6kCuw.png
---

#### `Promise`和`generator`的语法糖

async-await 是建立在 promise机制之上的，并不能取代其地位。

<!-- more -->

#### 基本语法

```javascript
async function demo() {
    let result = await Math.random()
    console.log(result)
}

demo()
// 0.008326278736928927
// Promise {<resolved>: undefined}
```



#### 基本概念

async用来表示函数是异步的，**定义的函数会返回一个promise对象**，可以使用then方法添加回调函数。

```javascript
async function demo01() {
    return 123;
}

demo01().then(val => {
    console.log(val);// 123
});
```



await 可以理解为是 async wait 的简写。await 必须出现在 async 函数内部，不能单独使用。

```javascript
function notAsyncFunc() {
    await Math.random();
}
notAsyncFunc();//Uncaught SyntaxError: Unexpected identifier
```

await 后面可以跟任何的JS 表达式。虽然说 await 可以等很多类型的东西，**但是它最主要的意图是用来等待 Promise 对象的状态被 resolved**。如果await的是 promise对象会**造成异步函数停止执行并且等待 promise 的解决**,如果等的是正常的表达式则立即执行。

```javascript
function sleep(second) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(' enough sleep~');
        }, second);
    })
}
function normalFunc() {
    console.log('normalFunc');
}
async function awaitDemo() {
    await normalFunc();
    console.log('something, ~~');
    let result = await sleep(2000);
    console.log(result);// 两秒之后会被打印出来
}
awaitDemo();
// normalFunc
// VM4036:13 something, ~~
// VM4036:15  enough sleep~
```

##### 实例

举例说明啊，你有三个请求需要发生，第三个请求是依赖于第二个请求的解构第二个请求依赖于第一个请求的结果。若用 ES5实现会有3层的回调，若用Promise 实现至少需要3个then。一个是代码横向发展，另一个是纵向发展。

接下来是`async await`的实现。

```javascript
//我们仍然使用 setTimeout 来模拟异步请求
function sleep(second, param) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(param);
        }, second);
    })
}

async function test() {
    let result1 = await sleep(2000, 'req01');
    let result2 = await sleep(1000, 'req02' + result1);
    let result3 = await sleep(500, 'req03' + result2);
    console.log(`
        ${result3}
        ${result2}
        ${result1}
    `);
}

test();
//req03req02req01
//req02req01
//req01
```



#### 错误处理

```js
function sleep(second) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject('want to sleep~');
        }, second);
    })
}

async function errorDemo() {
    let result = await sleep(1000);
    console.log(result);
}
errorDemo();// VM706:11 Uncaught (in promise) want to sleep~

// 为了处理Promise.reject 的情况我们应该将代码块用 try catch 包裹一下
async function errorDemoSuper() {
    try {
        let result = await sleep(1000);
        console.log(result);
    } catch (err) {
        console.log(err);
    }
}

errorDemoSuper();// want to sleep~
// 有了 try catch 之后我们就能够拿到 Promise.reject 回来的数据了。
```



#### 注意并行处理

await若是等到promise还在pending的状态时，就会停止下来。

```javascript
function sleep(second) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('request done! ' + Math.random());
        }, second);
    })
}

async function bugDemo() {
    await sleep(1000);
    await sleep(1000);
    await sleep(1000);
    console.log('clear the loading~');
}

bugDemo();
```

loading 确实是等待请求都结束完才清除的。但是你认真的观察下浏览器的 timeline 请求是一个结束后再发另一个的（若观察效果请发真实的 ajax 请求）

那么，正常的处理是怎样的呢？

```javascript
async function correctDemo() {
    let p1 = sleep(1000);
    let p2 = sleep(1000);
    let p3 = sleep(1000);
    await Promise.all([p1, p2, p3]);
    console.log('clear the loading~');
}
correctDemo();// clear the loading~
```



#### await in for 循环

`await`必须在`async`函数的上下文中的。

```javascript
// 正常 for 循环
async function forDemo() {
    let arr = [1, 2, 3, 4, 5];
    for (let i = 0; i < arr.length; i ++) {
        await arr[i];
    }
}
forDemo();//正常输出
// 因为想要炫技把 for循环写成下面这样
async function forBugDemo() {
    let arr = [1, 2, 3, 4, 5];
    arr.forEach(item => {
        await item;
    });
}
forBugDemo();// Uncaught SyntaxError: Unexpected identifier
```

