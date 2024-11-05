---
title: 用typescript实现依赖注入模式
description: 介绍如何使用typescript来实现依赖注入功能，并考虑到各种情况，包括实现单例和处理循环依赖等
date: 2022-04-03 12:14:55
categories:
  - Coding
tags:
  - Typescript
  - 依赖注入
  - 单例模式
  - 设计模式
---

# 用 typescript 实现依赖注入模式

本篇文章将介绍如何使用 typescript 来实现依赖注入功能，并考虑到各种情况，包括实现单例和处理循环依赖等

## 概念

依赖注入的概念非常简单。它基于一个容器（Container），用于管理对象的创建和解决依赖关系。在使用依赖注入之前，我们需要定义一些“可注入”的类（Injectable），并使用“Inject”标记它们的依赖关系。现在，让我们来逐步实现这个过程。

## 实现

### Step.1: 创建 @injectable 装饰器

@injectable 可以将一个类标记为一个可注入类, 可以使用 reflect-metadata 库来实现这个装饰器

首先, 安装 reflect-metadata 库

```sh
npm i reflect-metadata
```

之后在引入这个库并定义 injectable 函数

```ts
import "reflect-metadata";

export function Injectable() {
  return function (taget: any) {
    Reflect.defineMetadata("injectable", true, target);
  };
}
```

这个装饰器使用 Reflect.defineMetadata 为被装饰的类添加了一个 injectable 的元数据标识

### step.2: 创建 @inject 装饰器

@inject 装饰器用于标记类的依赖关系

```ts
import "reflect-metadata";

export function Inject() {
  return function (target: any, propertyKey: string) {
    const type = Reflect.getMetadata("design:type", target, propertyKey);
    Reflect.defineMetadata("inject", type, target, propertyKey);
  };
}
```

这个装饰器使用 Reflect.getMetadata 获取标记属性的类型, 随后用 Reflect.defineMetadata 将类型信息储存到元数据中

### step.3: 实现容器类

容器类 Container 用于管理类的实例的创建和解决依赖关系, 需要支持以下功能:

1. 注册标记为 @injectable 的类
2. 处理依赖关系
3. 创建单例
4. 解决循环依赖

```ts
import "reflect-metadata";

class Container {
  private instances: Map<Constructor, any> = new Map(); // 单例
  private resolvingQueue: Set<Constructor> = new Set(); // 解析队列

  register<T>(cls: Constructor<T>) {
    const injectable = Reflect.getMetadata("injectable", cls);
    if (!injectable) {
      throw new Error(`Class ${cls.name} is not marked with @injectable`);
    }
    this.instances.set(cls, null);
  }

  resolve<T>(cls: Constructor<T>): T {
    if (this.resolvingQueue.has(cls)) {
      // 当解析队列中包含这个类时, 表示出现循环依赖, 抛出错误
      throw new Error(`Circular dependency detected for class ${cls.name}`);
    }

    const instance = this.instances.get(cls);
    if (instance !== null) {
      // 当存在已解析的实例时, 则返回该实例, 实现单例模式
      return instance;
    }

    this.resolvingQueue.add(cls);

    const injectParams = Reflect.getMetadata("design:paramtypes", cls) || []; // 获取构造函数的参数类型列表
    const resolvedParams = injectParams.map((param) => this.resolve(param)); // 递归获取参数类的单例

    const newInstance = new cls(...resolvedParams);
    this.instances.set(cls, newInstance);

    this.resolvingQueue.delete(cls);

    return newInstance;
  }
}
```

这个 Container 类有两个主要方法: register 用于注册可注入的类, resolve 用于处理依赖关系, 并且维护单例和检查循环依赖

## 示例

将我们的依赖注入模块作为一个库引入, 来写一个简单示例

```ts
import { Injectable, Inject, Container } from "my-di-lib";

@Injectable()
class Foo {
  constructor(@Inject() public bar: Bar) {}
}

@Injectable()
class Bar {}

const container = new Container();

container.register(Foo);
container.register(Bar);

const foo = container.resolve(Foo);
foo instanceof Foo; // true
foo.bar instanceof Bar; // true
```
