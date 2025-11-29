---
title: Async/Await 与 Generator：关系深度解析与实现原理
description: 阐述Async/Await 与 Generator之间的底层联系，并展示如何通过 Generator 手动实现一个 async/await 的执行器（Executor）
date: 2023-01-21 16:13:28
categories:
  - Coding
tags:
  - Async
  - Await
  - Generator
---

# Async/Await 与 Generator：关系深度解析与实现原理

阐述 Async/Await 与 Generator 之间的底层联系，并展示如何通过 Generator 手动实现一个 async/await 的执行器（Executor）

## 1. 概述

在 JavaScript 异步编程的演进史中，`async/await` (ES2017) 被公认为异步操作的终极解决方案。然而，它并非凭空创造的新语法，而是基于 **Generator (生成器)** (ES2015) 和 **Promise** 的语法糖。

本文档旨在阐述两者之间的底层联系，并展示如何通过 Generator 手动实现一个 `async/await` 的执行器（Executor）。

## 2. 核心概念对比

### 2.1 Generator (生成器)

Generator 函数是 ES6 提供的一种异步编程解决方案。它是一个状态机，封装了多个内部状态。

- **特征**：函数名前带 `*`，内部使用 `yield` 暂停执行。
- **控制权**：执行权完全交给调用者。调用者需要手动调用 `iterator.next()` 来恢复执行。
- **数据交换**：`yield` 向外输出数据，`next(val)` 向内输入数据。

### 2.2 Async/Await

Async 函数是 Generator 函数的语法糖。

- **特征**：使用 `async` 关键字，内部使用 `await` 等待 Promise。
- **控制权**：自带执行器（Auto Executor）。一旦开始执行，就会自动运行直到结束。
- **本质**：**Async/Await = Generator + 自动执行器 (Auto Runner)**

## 3. 关系映射

我们可以将 `async/await` 的行为直接映射到 Generator 上：

| Async/Await 语法               | Generator 对应实现             | 作用                                      |
| :----------------------------- | :----------------------------- | :---------------------------------------- |
| `async function foo() {...}`   | `function* foo() {...}`        | 定义一个可暂停/恢复的函数                 |
| `const result = await promise` | `const result = yield promise` | 暂停执行，等待 Promise 解决，并将结果传回 |
| (自动执行)                     | `co(foo)` 或 自定义 runner     | 自动调用 `next`，处理 Promise 链          |

## 4. 原理深度解析

`async` 函数的执行过程实际上是一个**自动化的迭代过程**：

1.  当遇到 `await` (即 `yield`) 时，函数暂停。
2.  执行器拿到 `yield` 出来的 Promise。
3.  执行器等待该 Promise 状态变为 `resolved`。
4.  Promise 完成后，执行器将结果通过 `iterator.next(result)` 传回函数体，并恢复执行。
5.  如果 Promise 报错，执行器通过 `iterator.throw(error)` 抛出异常供函数内部 `try-catch` 捕获。
6.  重复上述步骤，直到生成器状态变为 `done: true`。

## 5. 实现自定义 Async/Await

为了证明上述关系，我们将实现一个名为 `run` (或通常称为 `asyncToGenerator`) 的高阶函数。它接收一个 Generator 函数，并返回一个 Promise。

### 5.1 核心实现代码

```javascript
/**
 * 自动执行器：模拟 async/await 的行为
 * @param {GeneratorFunction} genFunc - 生成器函数
 * @returns {Promise}
 */
function run(genFunc) {
  return new Promise((resolve, reject) => {
    // 1. 获取迭代器对象
    const gen = genFunc();

    // 2. 定义自动执行的步进函数
    function step(nextF) {
      let next;
      try {
        // 执行迭代器的 next() 或 throw()
        next = nextF();
      } catch (e) {
        // 如果生成器内部报错且未捕获，则 reject 外部 Promise
        return reject(e);
      }

      // 3. 判断生成器是否结束
      if (next.done) {
        // 如果结束，resolve 最终返回值
        return resolve(next.value);
      }

      // 4. 处理 yield 出来的值
      // 使用 Promise.resolve 包装，确保 yield 出来的即使是原始值也能处理
      Promise.resolve(next.value).then(
        (val) => {
          // Promise 成功：将结果传回生成器，并继续下一步
          step(() => gen.next(val));
        },
        (err) => {
          // Promise 失败：将错误抛回生成器，并继续下一步
          step(() => gen.throw(err));
        }
      );
    }

    // 5. 启动执行
    step(() => gen.next());
  });
}
```

### 5.2 代码逻辑拆解

1.  **返回 Promise**：`async` 函数总是返回一个 Promise，所以我们的包装函数也必须返回 Promise。
2.  **生成迭代器**：调用 `genFunc()` 得到迭代器 `gen`，此时代码并未开始执行。
3.  **Step 递归**：
    - 调用 `gen.next()` 获取 `{ value, done }`。
    - 检查 `done`：如果为 `true`，说明函数执行完毕，`resolve` 最终结果。
    - 如果为 `false`，说明遇到 `yield`（即 `await`）。
4.  **Promise 处理**：
    - 获取 `value`（通常是一个 Promise）。
    - 使用 `Promise.resolve(value)` 确保它是一个 Promise 对象。
    - `.then()` 成功回调中，递归调用 `step` 并传入 `gen.next(val)`，相当于把异步结果赋值给 `const res = yield ...` 左边的变量。
    - `.catch()` 失败回调中，递归调用 `step` 并传入 `gen.throw(err)`，这会触发生成器内部的 `try...catch`。

## 6. 实战演示

下面对比原生的 `async/await` 和我们使用 `Generator` + `run` 实现的版本。

### 6.1 模拟异步任务

```javascript
const getData = (n) => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(n * 2), 1000);
  });
};
```

### 6.2 方式对比

#### 方式 A: 原生 Async/Await

```javascript
async function mainAsync() {
  try {
    const data1 = await getData(10);
    const data2 = await getData(data1);
    console.log(`Async result: ${data2}`); // 输出 40
    return data2;
  } catch (error) {
    console.error(error);
  }
}

mainAsync();
```

#### 方式 B: Generator + 自定义运行器 (run)

```javascript
function* mainGen() {
  try {
    // yield 等同于 await
    const data1 = yield getData(10);
    const data2 = yield getData(data1);
    console.log(`Generator result: ${data2}`); // 输出 40
    return data2;
  } catch (error) {
    console.error(error);
  }
}

// 使用我们实现的 run 函数执行
run(mainGen).then((res) => console.log("Done:", res));
```

## 7. 异常处理机制

Generator 的一个强大之处在于 `iterator.throw()` 方法。这使得异步代码的错误处理可以像同步代码一样使用 `try...catch`。

在我们的 `run` 函数实现中：

```javascript
// 当 Promise 被 reject 时
(err) => {
  // 我们在生成器内部抛出错误
  step(() => gen.throw(err));
};
```

这一步操作会让代码执行指针回到 Generator 函数内部暂停的那一行（即 `yield` 处），并抛出一个异常。如果 Generator 函数内部包裹了 `try...catch`，这个异常就会被捕获，流程进入 `catch` 块。这与 `async/await` 的行为完全一致。

## 8. 总结

1.  **本质**：`async/await` 只是 Generator 函数的语法糖，并非底层新能力。
2.  **区别**：
    - Generator 需要手动控制迭代或配合 `co` 等库使用。
    - Async 函数内置了执行器，语义更清晰，兼容性更好。
3.  **转换**：任何 `async` 函数都可以被重写为 `Generator` 函数配合一个自动执行器。理解这一转换过程对于深入掌握 JavaScript 异步编程模型至关重要。
