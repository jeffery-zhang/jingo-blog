---
title: Nodejs 事件循环
description: 详细介绍 Nodejs 中的 事件循环机制
date: 2022-04-16 14:21:28
categories:
  - Web
tags:
  - Nodejs
  - 事件循环
  - 微任务
  - 定时器
---

# Nodejs 事件循环

详细介绍 Nodejs 中的 事件循环机制

## 事件循环经典题目

```js
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
console.log("script start");
setTimeout(function () {
  console.log("setTimeout0");
}, 0);
setTimeout(function () {
  console.log("setTimeout3");
}, 3);
setImmediate(() => console.log("setImmediate"));
process.nextTick(() => console.log("nextTick"));
async1();
new Promise(function (resolve) {
  console.log("promise1");
  resolve();
  console.log("promise2");
}).then(function () {
  console.log("promise3");
});
console.log("script end");
```

该题目涉及 Nodejs 中所有事件循环相关概念, 弄清楚这个顺序则对于 Nodejs 的事件循环机制就有一个清晰的了解了, 这段代码的输出结果为:

```sh
script start
async1 start
async2
promise1
promise2
script end
nextTick
async1 end
promise3
setTimeout0
setImmediate
setTimeout3
```

## 异步任务

异步任务有两种:

1. 追加在本轮循环的异步任务
2. 追加在次轮循环的异步任务

本轮循环一定早于次轮循环执行

Nodejs 规定 process.nextTick 和 promise 的回调函数追加在本轮循环, 同步任务执行完毕后就会开始执行他们, 而 setTimeout, setInterval 和 setImmediate 等 timers 的回调函数则追加到次轮循环

### process.nextTick()

Nodejs 执行完所有同步任务, 接下来就会执行 process.nextTick 的任务队列, 属于本轮循环

### 微任务(microtask)

promise 的回调函数会进入异步任务里的微任务队列

微任务队列追加在 process.nextTick 之后, 也属于本轮循环

```js
Promise.resolve().then(() => console.log(1));
process.nextTick(() => console.log(2));
```

以上代码始终先输出 2 再输出 1

```js
process.nextTick(() => console.log(1));
Promise.resolve().then(() => console.log(2));
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
```

输出结果 1, 3, 2, 4

开启 process.nextTick 任务会直接进入 nextTickQueue 中, 开启 promise 回调任务会直接进入 microTaskQueue 中, 只有前一个队列清空后才会执行下一个队列

### async 和 promise

async 函数返回一个 promise 对象, 当函数执行时是作为一个同步函数执行, 直到遇到 await, 就会先返回, 等到执行微任务阶段完成这个异步任务, 再执行函数体内后续操作

而在 promise 本身函数体中的代码在 resolve 之后依然可以执行, 因为在 then 回调之前, 该函数体内还是属于同步任务阶段, 正规的写法应该不要在 resolve 或 reject 后执行任何操作

## 事件循环阶段

1. timers 阶段

- 此阶段包括 setTimeout 和 setInterval

2. IO callbacks

- 大部分回调事件, 普通的 callback

3. poll 阶段

- 网络连接, 读取文件等操作

4. check 阶段

- setImmediate

5. close 阶段

- 一些 close 回调, 如 socket.on('close', ...) 等

### 开启事件循环

Nodejs 开始执行脚本时, 会先进行事件循环初始化, 此时还没有开始事件循环, 会优先处理以下任务:

1. 同步任务
2. 发出异步请求
3. 规划定时器生效时间
4. 执行 process.nextTick 回调
5. 开始事件循环

### setTimeout 和 setImmediate

由于 setTimeout 属于 timers 阶段, setImmediate 属于 check 阶段, 所以 setTimeout 始终早于 setImmediate 执行

```js
setTimeout(() => console.log(1));
setImmediate(() => console.log(2));
```

理论上, 这段代码会先输出 1 再输出 2, 但有时也会先输出 2 再输出 1

因为 setTimeout 第二个参数缺省值为 0, 但 Nodejs 做不到 0 毫秒执行回调函数, 至少也要 1 毫秒, 所以 setTimeout(..., 0) 等于 setTimeout(..., 1)

基于系统当前状态, 进入事件循环时可能不到 1 毫秒也可能超过 1 毫秒, 如果不到 1 毫秒, 则会先进入 check 阶段, 就会先执行 setImmediate 回调函数

但是如果在 I/O callbacks 阶段执行上述代码, 则会先执行 check 再执行 timers

```js
const fs = require("fs");

fs.readFile("aaa.json", () => {
  setTimeout(() => console.log(1));
  setImmediate(() => console.log(2));
});
```

上述代码必然先输出 2 再输出 1
