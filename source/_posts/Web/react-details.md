---
title: React 细节整理
description: 实际使用中遇到的一些 React 操作的细节和问题的记录
date: 2022-05-01 12:14:51
categories:
  - Web
tags:
  - React
  - React 细节
---

# React 细节整理

实际使用中遇到的一些 React 操作的细节和问题的记录

## 为什么开发环境中在挂在组件时使用 useEffect 会触发两次副作用

A: React 18 开始, 在 development 模式, 且使用了 strict mode 情况下, 挂载组件的 useEffect 会执行两次, 为了方便开发者模拟挂载后立即卸载组件, 可以提前发现重复挂载是否出现 bug

## 组件的 key 有什么作用

A: key 涉及到组件树的 diff 算法, diff 有如下规则:

1. 当元素类型变化时，会销毁重建
2. 当元素类型不变时，对比属性
3. 当组件元素类型不变时，通过 props 递归判断子节点
4. 递归对比子节点，当子节点是列表时，通过 key 和 props 来判断。若 key 一致，则进行更新，若 key 不一致，就销毁重建

key 的取值分为三种: 不定值, 索引值, 唯一值

1. 不定值: 如 Math.random(), 在列表渲染中如果修改列表值或者仅仅只是 setState 一个一样的列表, 也会导致真实 dom 被销毁
2. 索引值: 若使用列表索引作为 key, 在列表顺序被打乱的情况下不会销毁真实 dom, 但可能会导致非受控组件(如 input)互相篡改导致无法预期的变化
3. 唯一值: 推荐使用唯一值作为 key, 以保证 key 的唯一性和确定性

## useMemo 和 memo 的区别

useMemo 和 memo 都可用于缓存组件, 区别在于:

1. useMemo 可以缓存状态, memo 是一个 HOC, 只能缓存组件
2. useMemo 需要声明依赖项, memo 默认依赖于组件的 props

## 组件缓存方式

1. 使用 React.memo 或 useMemo 来对组件进行缓存
2. 子组件通过 props.children 来挂载, 能够保证子组件不更新, 这种方式传递给一个组件的子组件实际上是属于父组件的子组件, 所以当前组件更新不会触发这个子组件更新

## useLayoutEffect 与 useEffect

useLayoutEffect 与 useEffect 作用和语法都一样, 区别只有执行时机:

1. useEffect 执行在 React 渲染和提交之后, 是异步执行的, 可能会让界面的改动在下一次绘制才生效
2. useLayoutEffect 执行在浏览器绘制之前, 是同步执行的, 会阻塞渲染, 等价于 ComponentDidMount (源码中有调用实例的 ComponentDidMount 钩子), 并且在 ssr 模式下不能使用

总结:

1. 优先使用 useEffect, 因为它是异步不会阻塞渲染
2. 会影响渲染的操作可以放到 useLayoutEffect 中, 避免闪烁
3. 不能在 ssr 中使用 useLayoutEffect

## useCallback 作用

useCallback 可以缓存函数, 是对于 useMemo 功能的特化

### 使用场景

想象一个场景, 当有一个 React.memo 创建的子组件, 它的 prop 有一个值为函数, 来自父组件传递, 而这个函数是定义在父组件中的, 那么父组件每次渲染时, 都会创建一个新的函数实例导致子组件不必要的重新渲染, 而 useCallback 就是为了解决这个问题

useCallback 不是总会带来性能提升, 应该避免过度优化

应该使用的场景:

1. 子组件的性能优化: 当将一个函数传递给已经用 React.memo 优化过的子组件时, 使用 useCallback 可以确保子组件不会因为父组件重新渲染导致子组件不必要的重新渲染
2. Hook 依赖: 若函数被作为其他 hook 的依赖项, 使用 useCallback 可确保函数的稳定性, 可以避免不必要的副作用
3. 复杂计算和频繁的重新渲染: 如果组件涉及频繁的操作和反馈, 使用 useCallback 可以避免性能问题

不应使用的场景:

1. 不涉及传参和作为依赖的情况
2. 简单组件
3. 过度优化: 通常函数组件的重新渲染不会带来明显的性能问题, useCallback 没有必要

## useTransition

### 使用

```js
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setPage("/about");
});
```

### 注意点

1. useTransition 在 react 18 中加入, 需要通过 React.createRoot(root).render(<app />) 开启并发模式才能使用
2. startTransition 中的函数必须是同步的
3. 被 useTransition 包裹的同一个状态多次更新, 只会渲染最后一个, 前面的都算中断

## useDefferdValue

### 使用

```js
const [value, setValue] = useState();
const deferredText = useDeferredValue(value);
```

### 特性

类似于 useTransition 是可中断的单个状态, 会在其他紧急任务完毕后才执行, 并且每次只渲染最新的结果

### 与节流防抖的区别

1. 主要应用于优化界面的渲染, 不能终止如网络请求等操作
2. 不用设置固定的延迟时间, 会自动在空闲时间执行

## 父组件获取子组件的状态和方法的方式

1. 父组件将 ref 传递给子组件, 子组件用 forwardRef 包裹, 这样可以获取到子组件的元素
2. 在子组件中使用 useImperativeHandle 返回定义好的属性和方法, 并使用 forwardRef 包裹, 这样父组件就能通过 ref 调用子组件定义好的属性和方法

## useEffect 依赖对象深层比较

useEffect 对引用类型的依赖比较是浅比较(Object.is(obj1, obj2) 进行比较), 只要对象指针变化了就会触发副作用, 如果要深层比较对象字面量需要自定义 hooks, 如:

```ts
import { isEqual } from "lodash";

/**
 * @param effect 副作用函数
 * @param deps 依赖项, 可以是引用类型的值
 * @param compare 比较函数, 用于对象深层比较
 */
const useDeepCompareEffect = (effect, deps, compare) => {
  // 如果没传比较函数, 则默认使用lodash的isEqual
  if (!compare) compare = isEqual;
  // 标记是否改变
  const signal = useRef<number>(0);
  const memoizedDeps = useRef<any>([]);
  if (deps === undefined || !compare(memoizedDeps.current, deps)) {
    signal.current++;
  }
  // 缓存当前deps
  memoizedDeps.current = deps;
  useEffect(effect, [signal.current]);
};
```

## 高阶组件使用场景

1. 复用逻辑: HOC 可以帮助我们在组件之间复用逻辑, 避免重复代码, 如带 loading 状态的多个组件可以将 loading 逻辑放入 HOC 来避免在每个组件都声明 loading 状态
2. 修改 props: HOC 可以修改传递给组件的 props, 从而改变组件的行为, 如利用 HOC 根据权限显示或隐藏某些部分
3. 条件渲染: 可以根据条件来确定是渲染传入的组件还是其他组件
4. 提供额外功能: 可以为组件提供额外的功能, 如错误处理, 数据处理, 性能监控等
