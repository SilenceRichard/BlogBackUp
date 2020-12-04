---
title: Redux学习记录
date: 2020-04-02 14:38:54
tags:
  - 读书笔记
---

> 参考博文 [Redux 中文文档](https://cn.redux.js.org/)

# 什么时候使用 Redux

- 你有相当大量的、随时间变化的数据
- 你的 state 需要有一个单一可靠数据来源
- 你觉得把所有 state 放在最顶层组件中已经无法满足需求了

# 依赖

```bash
yarn add redux
yarn add react-redux
yarn add redux-devtools -D
```

# 要点

四个关键词： **store, state, action, reducers**

所有状态（state）存储在一个单一的仓库里 (store)，利用 action 触发改变 state

- action 一个普通的 javascript 对象

```js
// action 示例
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

- 为了描述 action 如何改变 state , 需要编写 reducers

# 三大原则

- ## 单一数据源
- ## STATE 只读

唯一改变 state 的方法是触发 action,对 state 的所有修改都是集中化，顺序化的

- ## 使用纯函数执行修改

与 flux 不同，redux 依赖纯函数替代事件处理器（无 dispatcher 概念），redux 设想你永远不会变动你的数据

# Action

> Action 是把数据从应用（译者注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

1.  我们约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作,通常 type 为字符串类型

2.  action 创建函数

    使 action 更具通用性

    ```js
    function addTodo(text) {
      return {
        type: ADD_TODO,
        text
      };
    }
    ```

3.  redux 把 action 创建函数的结果传给 dispatch()方法即可发起一次 dispatch 过程

    ```js
    dispatch(addTodo(text))
    dispatch(completeTodo(index))
    ```

```js
import { createStore } from "redux";

/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() => console.log(store.getState()));

// 改变内部 state 惟一方法是 dispatch 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: "INCREMENT" });
// 1
store.dispatch({ type: "INCREMENT" });
// 2
store.dispatch({ type: "DECREMENT" });
// 1
```
