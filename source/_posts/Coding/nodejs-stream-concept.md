---
title: Nodejs 中的 Stream
description: 详细介绍 Nodejs 中的 Stream(流) 的概念
date: 2022-04-16 14:21:28
categories:
  - Web
tags:
  - Nodejs
  - Stream
  - I/O
  - Http
  - Readdable
  - Writable
---

# Nodejs 中的 Stream

详细介绍 Nodejs 中的 Stream(流) 的概念

## Stream

Stream 是一种抽象的数据结构, 是 Nodejs 中的一个抽象的接口, 许多对象都实现了这个接口, 如 http 服务器的 request 和 response 等

## 为什么要使用 Stream

在 Nodejs 中处理大文件时, 使用 fs.readFileSync 或 fs.writeFileSync 等方法会将文件整体写入内存, 而流的作用可以把这些数据拆分, 每次只写入一小部分数据, 如:

```js
var fs = require("fs");
var readStream = fs.createReadStream("a.mp4"); // 创建可读流
var writeStream = fs.createWriteStream("b.mp4"); // 创建可写流

readStream.on("data", function (chunk) {
  // 当有数据流出时，写入数据
  writeStream.write(chunk);
});

readStream.on("end", function () {
  // 当没有数据时，关闭数据流
  writeStream.end();
});
```

当然这样写还是有问题, 读取流的速度总是快于写入流, 每次读取都触发写入可能会导致写入不能跟上读取速度, 而造成写入滞塞, 应对这个问题, 读取流还包含一个 pipe 方法可以将读取事件临时暂停, 等待写入流程完成:

```js
fs.createReadStream('a.mp4').pipe(fs.createWriteStream('b.mp4));
// pipe自动调用了data,end等事件
```

## Stream 来源 source

Stream 的常见来源方式有三种：

1. 从控制台输入
2. http 请求中的 request
3. 读取文件

## Stream 输出 dest

Stream 的常见输出方式有三种：

1. 输出控制台
2. http 请求中的 response
3. 写入文件

## Stream 管道 pipe

在 source 和 dest 之间有一个连接的管道 pipe,它的基本语法是 source.pipe(dest)，source 和 dest 就是通过 pipe 连接，让数据从 source 流向了 dest

## Stream 应用场景

Stream 的应用场景主要就是处理 IO 操作，而 http 请求和文件操作都属于 IO 操作。Stream 的本质——由于一次性 IO 操作过大，硬件开销太多，影响软件运行效率，因此将 IO 分批分段进行操作，让数据像水管一样流动起来，直到流动完成，也就是操作完成

## 继承自 Stream 的类

以下 4 个类继承自 Stream:

1. Readable 可读流
2. Writable 可写流
3. Duplex 可读可写流
4. Transform 在读写过程中可以修改和变换数据的 Duplex 流
