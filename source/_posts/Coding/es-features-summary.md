---
title: ES6+ 知识点整理
description: 介绍ES6及以上版本的新特性等知识点的整理笔记
date: 2022-06-01 10:13:22
categories:
  - Coding
tags:
  - ES6
  - ES2020
  - EcmaScript
---

# ES6+ 知识点整理

介绍 ES6 及以上版本的新特性等知识点的整理笔记

## Promise

通过 new Promise(function) 创建 promise 对象时必须传回调函数, 否则会报错

### Promise 三种状态

1. pending 初始状态, 处理中状态
2. fulfilled 被解决状态, 调用 then 回调函数
3. rejected 被拒绝状态, 调用 catch 回调函数

### Promise 静态方法

1. Promise.all 将多个 Promise 包装成一个, 若全部 fulfilled 则 resolve 所有 Promise, 若有一个失败则 reject
2. Promise.race 将多个 Promise 包装成一个, 若有一个 resolve 或 reject 则返回最快完成的那个 promise 实例
3. Promise.allSettled 将多个 Promise 包装成一个, 等所有 Promise 实例的状态都发生变更就执行 resolve, 无论这些 promise 的状态是成功还是失败
4. Promise.any 将多个 Promise 包装成一个, 若有一个 resolve 则返回最快完成的那个 promise 实例, 若所有都失败则 reject
5. Promise.resolve
6. Promise.reject

### Promise 和 async await

Async 是一个函数关键字, 声明一个函数是异步函数, 将返回一个 Promise 对象

Await 是一个运算符, 只能用于 async 函数中(也可以在代码顶层使用), 用于等待一个异步解析, 并返回解析完成后的值, 会暂停 async 函数的执行, 直到异步解析完毕后再执行后续代码

Promise 和 async/await 的使用场景:

1. 如果有很多连续的依赖于前一个异步操作结果的异步操作, 则可以使用 async/await, 这样更接近同步代码, 避免回调地狱
2. 如果需要执行多个并行的异步操作并在全部处理完后再处理结果, 则 Promise 更合适

## Symbol

新的数据类型, 声明一个不可重复的值, 在对象中可通过 Object.getOwnPropertySymbols 或 Reflect.ownKeys 遍历出来

若想得到两个相等的 Symbol 则需要用 Symbol.for(key)

```js
var s1 = Symbol.for("123");
var s2 = Symbol.for("123");

s1 === s2; // true
```

## var, let, const

1. var 定义在函数内则作用域仅在函数内, 若定义在函数外(包括 if 或 for 等代码块内), 则是全局作用域, 而 let 和 const 是块作用域
2. var 变量可以在其作用域内更新和重新声明；let 变量可以更新但不能重新声明；const 变量既不能更新也不能重新声明
3. 它们都被提升到了作用域的顶部, 但是, var 变量是用 undefined 初始化的, 而 let 和 const 变量不会被初始化
4. var 和 let 可以在不初始化的情况下声明, 而 const 必须在声明时初始化

## Set 和 Map

Set 是由一组无序且唯一(即不能重复)的项组成的, 可以想象成集合是一个既没有重复元素, 也没有顺序概念的数组

Map 类似于对象, 也是键值对的集合, 但是“键”的范围不限于字符串, 各种类型的值(包括对象)都可以当作键, 是一种更完善的 Hash 结构实现. 如果你需要“键值对”的数据结构, Map 比 Object 更合适

共同点: 集合, 字典可以存储不重复的值
不同点: 集合是以[值, 值]的形式存储元素, 字典是以[键, 值]的形式存储

## Proxy

可以给目标对象定义一个关联的代理对象, 而这个代理对象可以作为抽象的目标对象来使用, 在对目标对象的各种操作影响目标对象之前, 可以在代理对象中对这些操作加以控制. 代理对象可以作为目标对象的替身, 但又完全独立于目标对象

## 动态导入和顶层 await

ES2020 原生支持模块导入, 导入的模块不会污染全局命名空间

```js
if (conditon) {
  const module = await import("./myModule.js");
}
```

并且现在可以在顶级作用域中直接使用 await

## 空值合并

?? 运算符判断变量为 null 或 undefined

## globalThis

不论在什么环境下, globalThis 始终引用全局对象, 在浏览器是 window, 在 nodejs 是 global, 在 web worker 是 self

## Class Fields

类中使用\#开头可以声明 private 成员, 使用 static 可以声明静态成员, 同时可以在类中用 in 关键字判断实例中是否有私有字段

```js
class Person {
  #age = 50;
  name = "King";
  static gender = "male";
  static isPrivate(obj) {
    console.log(#age in obj);
  }
}

const person = new Person();
person.name; // King
person.#age; // Uncaught SyntaxError: Private field '#age' must be declared in an enclosing class
person.gender; // undefined
Person.gender; // male
Person.isPrivate(person); // true
```

## BigInt

JS 只能安全的表示-(2^53-1)至 2^53-1 范的值, 即 -9007199254740991 到 9007199254740991, 引入 BigInt 后可以在数字末尾加 n,表示更大的数字

```js
9007199254740991n + 1000n; // 9007199254741991n

// 但不能混用BigInt和number类型
9007199254740991n + 1000; // Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions
```
