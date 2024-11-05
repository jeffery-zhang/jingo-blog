---
title: Vue3 细节整理
description: 实际使用中遇到的一些 Vue3 操作的细节和问题的记录
date: 2022-05-02 16:24:31
categories:
  - Web
tags:
  - Vue3
  - Vue3 细节
---

# Vue3 细节整理

实际使用中遇到的一些 Vue3 操作的细节和问题的记录

## reactive 的局限

1. 只能用于对象类型(object, array, Map, Set 等), 不能用于原始值类型 (boolean, number, string 等), 因为 Proxy 只能作用于对象结构
2. 不能替换整个对象, 必须始终保持响应式对象的相同引用
3. 结构后对象属性会失去响应式特性

## ref 特性

1. ref 会将任意类型包装成一个 Proxy 对象, 通过内部的 value 属性挂载实际值
2. 如果传递原始数据类型, 则 ref 本身是一个 RefImpl 对象, 而 value 是原始值, 如果传递引用数据类型, 则 value 会包装成 Proxy 对象
3. ref 变量作为顶层属性时可以自动解包, 如 const count = ref(0), 在模板中应用时 <span>{count}<span/>, 而引用数据类型则不能自动解包, 如 const person = ref({ age: 2 }), 在模板中 <span>{person.age}<span/> 无法正确获取到值, 需要手动解包, 如 const { age } = person

## 父子组件执行顺序

父 setup -》 父 onBeforeMount =〉子 setup -》子 onBeforeMount -〉子 onMounted -》 父 onMounted

## watch 深层监听

1. 当传递一个响应式对象给 watch 而不是一个 getter 函数时, watch 会自动创建响应式对象的深层监听, 如:

```js
const obj = reactive({
  inner: {
    count: 10,
  },
});

const addObjCount = () => {
  obj.inner.count++;
};

// 此时修改 count 时会触发 watch 回调函数
watch(obj, (newValue, oldValue) => {
  console.log("新的 obj:", newValue.inner);
  console.log("旧的 obj:", oldValue.inner);
});

// 等价于
// watch(
//   () => obj,
//   (newValue, oldValue) => {
//     console.log("新的 obj:", newValue.inner);
//     console.log("旧的 obj:", oldValue.inner);
//   },
//   {
//     deep: true,
//   }
// );
```

此时绑定 addObjCount 函数到点击事件, 会发现每次点击都触发了 watch 回调函数执行, 而修改 watch 的参数为 getter 函数后, 则不会触发

```js
// 此时修改 count 时不会触发 watch 回调函数
watch(
  () => obj,
  (newValue, oldValue) => {
    console.log("新的 obj:", newValue.inner);
    console.log("旧的 obj:", oldValue.inner);
  }
);
```

## watch 和 watchEffect 区别

1. watch 需要指明监听对象, watchEffect 隐式监听回调函数中的响应式数据
2. watch 只有在数据变化时执行回调函数, watchEffect 会在响应式数据初始化时就执行回调函数

## watch / watchEffect 回调函数执行时机

默认情况下, 监听回调执行时机是在 vue 组件渲染到 dom 之前, 如果设置了 flush: 'post' 则是在渲染之后执行, 可以获取到更新后的 dom

## v-show 和 v-if 异同

共同点:

1. 控制元素是否显示在界面
2. 为 true 时会占据页面位置, 为 false 时不会占据
3. 都会导致回流(reflow)和重绘(repaint)

不同点:

1. v-show 无论何值都会渲染元素, 但使用 css display 属性来控制显示隐藏, v-if 为 false 则是直接不渲染元素
2. v-show 由 false 变为 true 不会触发生命周期, v-if 由 false 变为 true 会触发 create 和 mount 阶段的生命周期钩子, true 改为 false 则会触发 destroy 阶段的钩子
3. v-if 有更高的切换性能消耗, v-show 则是更高的初始渲染消耗

## v-for 中的 key

Vue 在处理更新同类型 vnode 的一组子节点的过程中, 为了减少 DOM 频繁创建和销毁的性能开销

假设要更新这样一个节点列表

![更新vue节点](/images/vue3-details-01.png)

1. 对没有 key 的子节点数组更新调用的是 patchUnkeyedChildren 这个方法, 核心是就地更新的策略. 它会通过对比新旧子节点数组的长度, 先以比较短的那部分长度为基准, 将新子节点的那一部分直接 patch 上去. 然后再判断, 如果是新子节点数组的长度更长, 就直接将新子节点数组剩余部分挂载(mount); 如果是新子节点数组更短, 就把旧子节点多出来的那部分给卸载掉(unmount). 所以如果子节点是组件或者有状态的 DOM 元素, 原有的状态会保留, 就会出现渲染不正确的问题

![更新不带key的节点](/images/vue3-details-02.png)

2. 有 key 的子节点更新是调用的 patchKeyedChildren, 这个函数就是大家熟悉的实现核心 diff 算法的地方, 大概流程就是同步头部节点, 同步尾部节点, 处理新增和删除的节点, 最后用求解最长递增子序列的方法去处理未知子序列. 是为了最大程度实现对已有节点的复用, 减少 DOM 操作的性能开销, 同时避免了就地更新带来的子节点状态错误的问题

![更新带key的节点](/images/vue3-details-03.png)

如果是用 v-for 去遍历常量或者子节点是诸如纯文本这类没有状态的节点, 是可以使用不加 key 的写法的。但是实际开发过程中更推荐统一加上 key, 能够实现更广泛场景的同时, 避免了可能发生的状态更新错误, 我们一般可以使用 ESlint 配置 key 为 v-for 的必需元素

通常在数据量大的列表等情况下推荐给元素加上 key, 可以有效减少 diff 的开销, 因为没有 key, 算法需要进入子节点深入比较变化, 加上 key 可以最大程度复用节点, 进行节点位置的移动而不是频繁销毁和创建新的节点

## vue3 中的宏命令

宏是一种特殊代码, 运行在编译阶段, 会被编译器转换成其他代码, 实际上是一种巧妙地字符串替换技巧, 根据功能不同, 转换后的代码也不同

### 为什么宏不用 import

import 的模块在运行时执行, 编译时执行的宏命令不需要 import

### 注意点

编译时只会处理 setup 顶层的宏, 被包裹在代码块中的宏不会被编译器转换, 导致在运行时执行这些代码时报错

## nextTick

在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法, 获取更新后的 DOM

### 用法

```js
import { createApp, nextTick } from "vue";
const app = createApp({
  setup() {
    const message = ref("Hello!");
    const changeMessage = async (newMessage) => {
      message.value = newMessage;
      // 这里获取DOM的value是旧值
      await nextTick();
      // nextTick 后获取DOM的value是更新后的值
      console.log("Now DOM is updated");
    };
  },
});
```

### 特性

1. 如果你想捕捉组件数据变化后 DOM 更新的时刻, 那么你需要使用 nextTick(callback) 函数
2. 它们的单个 callback 参数在 DOM 更新后立即被调用: 并且你可以保证获得与组件数据同步的最新 DOM
3. 或者, 如果你不向 nextTick() 提供回调参数, 那么这些函数将返回一个在 DOM 更新时被解析的 promise

## h 函数

返回一个"虚拟节点", 通常缩写为 VNode: 一个普通对象, 其中包含向 Vue 描述它应该在页面上呈现哪种节点的信息, 包括对任何子节点的描述, 用于手动编写 render

h 接受三个参数

1. type 元素类型
2. propsOrChildren 数据对象, 如 props, domProps, class, attrs, style 等
3. children 子节点

通过内部的 \_createVNode 方法生成 VNode 虚拟节点
