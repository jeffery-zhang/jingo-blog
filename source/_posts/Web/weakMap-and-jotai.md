---
title: WeakMap 与 Jotai 的实现原理
description: 详细讲解 ES6+ 中的 WeakMap 的作用和 Jotai 原子状态管理库的实现原理
date: 2022-10-03 14:01:06
categories:
  - Web
tags:
  - WeakMap
  - Jotai
---

# WeakMap 与 Jotai 的实现原理

详细讲解 ES6+ 中的 WeakMap 的作用和 Jotai 原子状态管理库的实现原理

这是一个非常深入且极具技术含量的问题。理解 `WeakMap` 是理解 Jotai 设计哲学的关键，甚至可以说是 Jotai 能够实现“原子化且无内存泄漏”的核心基石。

我们分两部分来讲：先彻底搞懂 **WeakMap**，再看 **Jotai** 如何利用它。

---

### 第一部分：WeakMap 深度解析

在 JavaScript 中，普通的 `Map` 和 `Object` 都会建立“强引用”。只要 Map 还在，Map 里的 Key 和 Value 就永远不会被垃圾回收（GC）。

`WeakMap` 是 ES6 引入的一种特殊 Map，它有两个核心特性：**Key 必须是对象**，以及**弱引用机制**。

#### 1. 什么是“弱引用” (Weak Reference)？

这是 WeakMap 的灵魂。

- **强引用**：如果不把这一根线剪断，垃圾回收器（GC）绝不敢回收这个对象。
- **弱引用**：这根线是“虚”的。GC 在检查内存时，**会无视 WeakMap 对 Key 的引用**。

**通俗比喻**：

- **普通 Map (强引用)**：你牵着一条狗（Key）。只要你手里的绳子（Map）还在，狗就不能跑，捕狗队（GC）也不能把狗抓走。
- **WeakMap (弱引用)**：你看着一条狗，但没用绳子拴住它。只要这只狗没有被其他人（其他变量）牵着，它随时可能跑掉或被捕狗队（GC）带走。一旦狗没了，你脑子里关于这条狗的记忆（Value）也就自动消失了。

#### 2. WeakMap 的特性

1.  **Key 必须是对象**（或者非全局注册的 Symbol）：不能是 `1`, `"string"`, `true` 这种原始值。因为原始值没有生命周期，无法被 GC，所以放进 WeakMap 没意义。
2.  **不可遍历 (Non-iterable)**：你不能调用 `keys()`, `values()`, `forEach()`，也不能知道 `size`。
    - **为什么？** 因为 GC 的运行时机是不确定的。可能上一秒 Key 还在，下一秒就被回收了。如果允许遍历，结果就是“薛定谔的 Map”，会导致逻辑混乱。
3.  **自动垃圾回收**：这是最重要的。一旦 Key 对象在 WeakMap 之外没有任何引用了，JS 引擎会自动把这个 Key 和对应的 Value 从 WeakMap 中移除。

---

### 第二部分：Jotai 如何使用 WeakMap

Jotai 的核心哲学是：**Atom 只是一个定义（Key），Store 才是存值的地方（WeakMap）。**

#### 1. Jotai 的数据模型

在 Redux 或 Zustand 中，状态是一个巨大的普通对象。但在 Jotai 中，状态被拆分了。

- **Atom (定义)**：当你写 `const countAtom = atom(0)` 时，`countAtom` 只是一个普通的配置对象，大概长这样：

  ```javascript
  // countAtom (Key)
  {
    init: 0,
    read: (get) => ...,
    write: (set) => ...
  }
  ```

  **注意：这里面没有存具体的值（如 1, 2, 3）！它只是一个“身份证”。**

- **Store (容器)**：Jotai 的 Store 内部维护了一个 `WeakMap` 来存储真实的状态值。
  ```typescript
  // 简化的 Store 结构
  const atomStateMap = new WeakMap<AtomConfig, AtomState>();
  ```

#### 2. 为什么 Jotai 必须用 WeakMap？

这是为了解决**内存泄漏**和支持**动态 Atom**。

想象一下，如果 Jotai 内部用的是普通的 `Map`：

```javascript
// ❌ 假设 Jotai 内部用的是普通 Map (强引用)
const globalStateMap = new Map();

function MyComponent() {
  // 场景：在组件内部动态创建 Atom
  const localAtom = useMemo(() => atom(0), []);

  // 挂载时，Jotai 把 localAtom 存入 Map
  // globalStateMap.set(localAtom, { value: 0 });

  return <div>...</div>;
}
```

**后果**：

1.  当 `MyComponent` 卸载时，`localAtom` 变量被销毁。
2.  但是！因为 `globalStateMap` 是强引用，它依然死死抓住 `localAtom` 这个对象不放。
3.  GC 无法回收 `localAtom`，也无法回收它对应的状态值 `{ value: 0 }`。
4.  如果这个组件反复挂载/卸载 1000 次，Map 里就会有 1000 个永远不用的僵尸条目。**这就是内存泄漏。**

**使用 WeakMap 的情况 (✅ Jotai 的做法)**：

```javascript
// ✅ Jotai 内部使用 WeakMap
const globalAtomStateMap = new WeakMap();

function MyComponent() {
  // 1. 创建 Atom 对象 (Key)
  const localAtom = useMemo(() => atom(0), []);

  // 2. 存入 WeakMap
  // globalAtomStateMap.set(localAtom, { value: 0 });
}
```

**结果**：

1.  当 `MyComponent` 卸载，`localAtom` 变量失去引用。
2.  JS 引擎发现 `localAtom` 对象只剩 `WeakMap` 里的引用了（那是弱引用）。
3.  GC 直接回收 `localAtom` 对象。
4.  **连带效应**：`globalAtomStateMap` 里的这条记录（Key 和 Value）自动消失。
5.  **完美无残留。**

#### 3. 模拟 Jotai 的底层实现代码

让我们写一个极简版的 Jotai Store，看看 WeakMap 是怎么工作的：

```javascript
// 极简版 Jotai Store
const createStore = () => {
  // 核心：用 WeakMap 存储状态
  // Key: atom 配置对象
  // Value: 具体的 state 值和监听器
  const atomStateMap = new WeakMap();

  const get = (atom) => {
    // 如果 WeakMap 里有，就取出来；没有就用 atom.init 初始化
    let atomState = atomStateMap.get(atom);
    if (!atomState) {
      atomState = { value: atom.init, listeners: new Set() };
      atomStateMap.set(atom, atomState); // 存入 WeakMap
    }
    return atomState.value;
  };

  const set = (atom, newValue) => {
    const atomState = atomStateMap.get(atom);
    if (atomState) {
      atomState.value = newValue;
      // 通知订阅者
      atomState.listeners.forEach((callback) => callback());
    }
  };

  const subscribe = (atom, callback) => {
    let atomState = atomStateMap.get(atom);
    if (!atomState) {
      atomState = { value: atom.init, listeners: new Set() };
      atomStateMap.set(atom, atomState);
    }
    atomState.listeners.add(callback);
    return () => atomState.listeners.delete(callback);
  };

  return { get, set, subscribe };
};

// --- 测试 ---

// 1. 创建 Store
const store = createStore();

// 2. 创建一个 Atom (这是一个对象)
let myAtom = { init: 0 };

// 3. 使用 Atom
store.set(myAtom, 100);
console.log(store.get(myAtom)); // 100

// 4. 模拟组件卸载，myAtom 失去引用
myAtom = null;

// 此时：
// 即使 store 还在，但由于 myAtom (Key) 已经没了，
// WeakMap 会在下一次 GC 时自动把存的 100 这个值清理掉。
```

### 总结

Jotai 使用 `WeakMap` 的精髓在于：

1.  **Atom 对象即是 Key**：利用对象引用的唯一性，不需要给 Atom 起字符串名字（解决了命名冲突问题）。
2.  **生命周期绑定**：Atom 状态的生命周期**严格绑定**于 Atom 定义对象的生命周期。
    - 如果是全局定义的 `const atom = ...`，它永远存在，状态也永远存在（类似 Redux）。
    - 如果是组件内定义的 `const atom = ...`，组件销毁 -> atom 对象销毁 -> 状态自动清理。

这种设计使得 Jotai 极其灵活，既可以做全局状态管理，也可以做组件级的局部状态管理，而完全不需要开发者手动去清理（`dispose`）内存。
