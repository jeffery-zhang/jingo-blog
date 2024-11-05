---
title: React Fiber
description: 详细介绍 React 中的 fiber 机制和原理
date: 2022-06-15 15:51:52
categories:
  - Web
tags:
  - React
  - Fiber
---

# React Fiber

详细介绍 React 中的 fiber 机制和原理

## 什么是 Fiber

Fiber 就是一个 js 对象, 用于描述一个 react 工作单元. 早期的 react 中是使用虚拟 DOM 来进行描述, 现在的 fiber 架构中则是用 fiber, fiber 可以理解为一个更强大的虚拟 DOM

一个简化的 fiber 对象如下:

```js
{
  type: 'h1',  // 组件类型
  key: null,   // React key
  props: { ... }, // 输入的props
  state: { ... }, // 组件的state (如果是class组件或带有state的function组件)
  child: Fiber | null,  // 第一个子元素的Fiber
  sibling: Fiber | null,  // 下一个兄弟元素的Fiber
  return: Fiber | null,  // 父元素的Fiber
  // ...其他属性
}
```

React 工作时, 会沿着 fiber 树形结构进行, 对比每个 fiber 旧的 props 和新的 props 来确定是否需要更新组件, 如果主线程有更重要的工作, 如响应用户输入, 则可以中断当前工作并返回主线程上的任务

## Fiber 结构

```js
function FiberNode(
  this: $FlowFixMe,
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode
) {
  // 基本属性
  this.tag = tag; // 描述此Fiber的启动模式的值（LegacyRoot = 0; ConcurrentRoot = 1）
  this.key = key; // React key
  this.elementType = null; // 描述React元素的类型。例如，对于JSX<App />，elementType是App
  this.type = null; // 组件类型
  this.stateNode = null; // 对于类组件，这是类的实例；对于DOM元素，它是对应的DOM节点。

  // Fiber链接
  this.return = null; // 指向父Fiber
  this.child = null; // 指向第一个子Fiber
  this.sibling = null; // 指向其兄弟Fiber
  this.index = 0; // 子Fiber中的索引位置

  this.ref = null; // 如果组件上有ref属性，则该属性指向它
  this.refCleanup = null; // 如果组件上的ref属性在更新中被删除或更改，此字段会用于追踪需要清理的旧ref

  // Props & State
  this.pendingProps = pendingProps; // 正在等待处理的新props
  this.memoizedProps = null; // 上一次渲染时的props
  this.updateQueue = null; // 一个队列，包含了该Fiber上的状态更新和副作用
  this.memoizedState = null; // 上一次渲染时的state
  this.dependencies = null; // 该Fiber订阅的上下文或其他资源的描述

  // 工作模式
  this.mode = mode; // 描述Fiber工作模式的标志（例如Concurrent模式、Blocking模式等）。

  // Effects
  this.flags = NoFlags; // 描述该Fiber发生的副作用的标志（十六进制的标识）
  this.subtreeFlags = NoFlags; // 描述该Fiber子树中发生的副作用的标志（十六进制的标识）
  this.deletions = null; // 在commit阶段要删除的子Fiber数组

  this.lanes = NoLanes; // 与React的并发模式有关的调度概念。
  this.childLanes = NoLanes; // 与React的并发模式有关的调度概念。

  this.alternate = null; // Current Tree和Work-in-progress (WIP) Tree的互相指向对方tree里的对应单元

  // 如果启用了性能分析
  if (enableProfilerTimer) {
    // ……
  }

  // 开发模式中
  if (__DEV__) {
    // ……
  }
}
```

## Fiber 工作原理

Fiber 工作原理最核心的点就是可以中断和恢复, 增强了 react 的并发性和响应性

工作原理中的几个关键点:

1. 单元工作: 每个 fiber 节点代表一个单元, 所有 fiber 节点构成一个 fiber 链表树, 使 react 可以细粒度控制节点行为
2. 链接属性: child, sibling, return 分别表示子节点, 兄弟节点和父节点, 构成了 fiber 之间的链接关系, 使 react 能够遍历 fiber 树来确定从哪里开始, 继续或停止工作
3. 双缓冲技术: react 在更新时会根据现有的 fiber 树 (current tree) 创建一个新的临时树 (work in progress (WIP) tree), WIP tree 包含了当前更新受影响的最高节点和其下所有子孙节点, WIP tree 在后台更新, current tree 则是显示在界面上的视图. WIP tree 更新完成后会复制其他节点并最终替换掉 current tree. 因为同时维护两个 fiber tree, 所以 react 可以随时进行比较, 中断或恢复等操作, 这种机制同时保证了渲染性能和 UI 稳定
4. State & props: memoizedProps, pendingProps, memoizedState 字段让 react 知道组件的上一个状态和即将应用的状态, 通过比较这些值, react 可以决定组件是否需要更新, 避免不必要的渲染
5. 副作用追踪: flags 和 subtreeFlags 标识 fiber 及其子树中需要执行的副作用, 如 DOM 更新, 生命周期调用等, react 会收集这些副作用, 在 commit 阶段一次性执行

## Fiber 工作流程

主要分为两个阶段:

### Reconciliation 调和

调和阶段确定了哪些部分的 UI 需要更新, 通过比较新的 props 和旧的 fiber 树来确定, 调和阶段可中断和恢复

调和阶段分为两个小阶段:

1. 创建与标记更新节点: beginWork, 这个阶段会进行判断 fiber 是否需要更新, 以及判断 fiber 子节点是更新还是复用, 随后执行执行 fiber 节点的调和(处理诸如新 fiber 的创建, 旧 fiber 的删除或现有 fiber 的更新). beginWork 完成后就会进入 completeWork 流程
2. 收集副作用列表: completeUnitOfWork & completeWork: completeUnitOfWork 负责遍历 fiber 节点, 同时记录有副作用的节点的关系, completeWork 在 completeUnitOfWork 中被调用, 主要用于记录 fiber 的副作用标志, 为子 fiber 创建链表以及根据 fiber 的 tag 进行不同的处理

### Commit 提交

提交阶段会通过遍历在 Reconciliation 阶段创建的副作用列表来更新 DOM 并执行收集到的副作用, 提交阶段不可中断

提交阶段分为三个小阶段:

1. 遍历副作用列表: commitBeforeMutationEffects, 遍历 fiber, 处理节点删除和确认节点在 before mutation 阶段是否有要处理的副作用
2. 正式提交: commitMutationEffects, 递归遍历 Fiber, 更新副作用节点
3. 处理 layout effects: commitLayoutEffects, 处理那些由 useLayoutEffect 创建的 layout effects
