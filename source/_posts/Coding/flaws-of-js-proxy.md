---
title: ES6 Proxy 对象特性和存在的缺陷
description: 详细介绍 ES6 中新加入的 Proxy 对象的特性和在使用中存在的缺陷
date: 2022-04-24 11:33:28
categories:
  - Coding
tags:
  - Proxy
  - 代理
  - ES6
  - Proxy缺陷
  - Reflect
---

# ES6 Proxy 对象特性和存在的缺陷

详细介绍 ES6 中新加入的 Proxy 对象的特性和在使用中存在的缺陷

## Proxy 实例

首先看看 Proxy 是如何代理一个对象的

```js
class Person {
  constructor(first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  introduce(other = "friend") {
    console.log(`Hello ${other}, my name is ${this.fullName}!`);
  }
}

const leo = new Person("Leo", "Messi");

const proxy = new Proxy(leo, {
  get(target, prop) {
    console.log(`Access: ${prop}`);
    return Reflect.get(target, prop);
  },
});

proxy.introduce();
```

上面的代码实例化了一个 Proxy 对象, 传递一个 Person 对象来作为被代理的对象, 这时, 调用 proxy 的 get 方法要做两件事:

1. 将正在检索的对象键打印出来
2. 使用 Reflect.get 将属性值从实例化的 Person 对象中取出来

上面的打印结果是:

```sh
Access: introduce
Access: fullName
Hello friend, my name is Leo Messi!
```

1. 在 js 的对象中,调用一个方法时, 必然先调用对象上的 get 方法, 所以第一条打印信息是 introduce
2. 此时, introduce 方法中的 this 指向 proxy 对象, 所以调用 this.fullName 会再次调用 proxy 的 get 方法打印第二条信息
3. 最后打印 introduce 方法的返回值

但为什么没有打印 firstName 和 lastName 呢, 当访问 fullName 时内部确实也访问了这两个变量的

在我们使用 Reflect.get 访问内部的 fullName 时, 因为 fullName 是一个属性, 它会调用属性描述符上的 get 方法, 此时的 this 在运行时已经指向了 target, 而 target 不是 proxy, 不会触发 proxy 中定义的 get 方法

为了完成 person 对象的全面代理, 则需要设置 Reflect 的第三个参数:

```js
...

const proxy = new Proxy(leo, {
  get(target, prop, receiver) {
    console.log(`Access: ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
});

proxy.introduce();
```

这个操作会将使用 Reflect.get 获取 taget 内部属性的 this 也指向调用者, 而调用者则是 proxy

打印结果如下:

```sh
Access: introduce
Access: fullName
Access: firstName
Access: lastName
Hello friend, my name is Leo Messi!
```

## Proxy 撤销

通过 Proxy.revocable 方法创建的代理对象是可撤销代理的对象, 这种代理可以被创建者禁用, 下面是一个示例:

```js
...

const { proxy, revoke } = Proxy.revocable(leo, {
  get(target, prop, receiver) {
    console.log(`Access: ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
});

proxy.introduce();
revoke()
proxy.introduce("boy");
```

其结果为:

```sh
Access: introduce
Access: fullName
Access: firstName
Access: lastName
Hello friend, my name is Leo Messi!
Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```

Proxy.revocable 会返回一个 revoke 方法, 调用后就可以撤销代理

## Proxy 的缺陷

### 兼容性

作为 ES6 的特性, Proxy 对于旧版浏览器支持不好, 即使使用 proxy-polyfill 也仅能部分支持 Proxy 的功能

在 github 上的 GoogleChrome/proxy-polyfill 库目前只支持以下 traps:

- get
- set
- apply
- constructor

### 性能

Proxy 的性能比 Promise 还差, 也差于 Object.defineProperty

### 不能安全代理私有成员

修改之前的例子:

```js
class Person {
  #firstName;
  #lastName;

  constructor(first, last) {
    this.#firstName = first;
    this.#lastName = last;
  }

  get firstName() {
    return this.#firstName;
  }

  get lastName() {
    return this.#lastName;
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  introduce(other = "friend") {
    console.log(`Hello ${other}, my name is ${this.fullName}!`);
  }
}

const leo = new Person("Leo", "Messi");

const proxy = new Proxy(leo, {
  get(target, prop) {
    console.log(`Access: ${prop}`);
    return Reflect.get(target, prop);
  },
});

proxy.introduce();
```

现在看看打印结果:

```sh
Access: introduce
Access: fullName
Hello friend, my name is Leo Messi!
```

貌似没有问题, 但如果给 Reflect 传入第三个参数, 则会出现问题:

```js
...

const proxy = new Proxy(leo, {
  get(target, prop, receiver) {
    console.log(`Access: ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
});

proxy.introduce();
```

打印结果为:

```sh
Access: introduce
Access: fullName
Access: firstName
Uncaught TypeError: Cannot read private member #firstName from an object whose class did not declare it
```

可以看到, 但传递 receiver 时, 调用 firstName 时 this 会指向 proxy, 而这个调用这个属性的 getter 时会指向一个私有变量, 私有变量不能在对象外获取, 则会出现这个问题

如果方法中使用了私有成员, 如:

```js
...
  introduce(other = "friend") {
    console.log(`Hello ${other}, my name is ${this.#firstName} ${this.#lastName}!`);
  }
```

则可以在 Proxy 的 getter 中再次改变 this 指向来用原有对象调用该方法:

```js
...

const proxy = new Proxy(leo, {
  get(target, prop, receiver) {
    console.log(`Access: ${prop}`);
    const value = Reflect.get(target, prop, receiver);
    return typeof value === 'function' ? value.bind(target) : value
  },
});

proxy.introduce();
```

打印结果为:

```sh
Access: introduce
Hello friend, my name is Leo Messi!
```

这样就能在调用使用了私有成员的方法时获取到正确的结果了
