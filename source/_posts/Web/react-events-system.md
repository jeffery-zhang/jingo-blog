---
title: React 事件机制原理
description: 详细介绍 React 中的合成事件机制和实现原理
date: 2022-06-15 10:31:22
categories:
  - Web
tags:
  - React
  - 事件
  - 原生事件
  - 合成事件
  - 事件机制
  - 事件系统
---

# React 事件机制原理

详细介绍 React 中的合成事件机制和实现原理

## 原生事件

浏览器基于 W3C 规范实现了一套标准化 DOM 事件, 基于 Event 类实现了常见的用户事件如 InputEvent, MouseEvent 等

事件发生时, 相关信息会储存在 Event 实例对象中, 包括 currentTarget, detail, target, preventDefault(), stopPropagation() 等属性方法, DOM 节点可以通过 addEventListener 和 removeEventListener 来添加或移除事件监听函数

## React 合成事件

React 在基于原生事件规范的前提下实现了合成事件(Synthetic Events), 合成事件与原生事件不是一一对应关系, 如 onChange 事件就由 change, click, input, keydown, keyup 等原生事件组成

## 事件机制

### 事件注册

使用 React.createRoot 创建 root 时, React 会调用 listenToAllSupportedEvents 方法对所有支持的原生事件进行监听, 该方法会收集并返回 allNativeEvents, 用于收集所有合成事件相关联的原生事件名, 并且这个收集动作在事件插件初始化阶段就完成了

随后对每个原生事件调用 addTrappedEventListener 函数, 函数内最终使用 addEventListener 方法对原生事件进行捕获或冒泡阶段的事件监听注册

```js
function addTrappedEventListener(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  isCapturePhaseListener: boolean
) {
  let listener = createEventListenerWrapperWithPriority(
    targetContainer,
    domEventName,
    eventSystemFlags
  );

  // ...

  if (isCapturePhaseListener) {
    addEventCaptureListener(targetContainer, domEventName, listener);
  } else {
    addEventBubbleListener(targetContainer, domEventName, listener);
  }
}
```

基于这个流程可知, 调用 React.createRoot 时就已经在 root 节点上初始化了所有原生事件的监听回调函数, 而不是在执行组件函数时才进行注册

### 事件触发

![react-events-system-01](/images/react-events-system-01.png)

在注册事件阶段调用的 addTrappedEventListener 方法中, 会使用 createEventListenerWrapperWithPriority 函数来创建事件回调. createEventListenerWrapperWithPriority 函数根据事件类型, 划分出若干个不同优先级的 dispathEvent. 事件回调最终都调用进 dispatchEvent 方法

触发一个事件的执行流程如下:

1. 原生事件触发后进入 dispathEvent 回调方法
2. attemptToDispatchEvent 方法根据该原生事件查找到当前 DOM 节点和映射的 Fiber 节点
3. 事件和 Fiber 等信息被派发给插件系统进行处理, 插件系统调用各插件暴露的 extractEvents 方法
4. accumulateSinglePhaseListeners 方法向上收集 Fiber 树上监听相关事件的其他回调函数, 构造合成事件并加入到派发队列 dispatchQueue 中
5. 调用 processDispatchQueue 方法, 基于捕获或冒泡阶段的标识, 按倒序或顺序执行 dispatchQueue 中的方法

## 总结

React 的事件处理机制可以分为两个阶段, React.createRoot 时在 root 节点上注册原生事件, 原生事件触发时模拟捕获, 目标和冒泡阶段派发合成事件. 通过这种机制, 冒泡的原生事件类型最多在 root 节点上注册一次, 节省了内存开销. 且 React 为不同类型的事件定义了不同的优先级, 让代码及时响应高优先级的用户交互, 提升用户体验

React 的合成事件在符合 W3C 规范的前提下抹平了不同浏览器的差异, 并且简化事件逻辑, 对关联事件进行合成
