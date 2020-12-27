---
title: Promise的实现
date: 2020-12-27 19:49:02
tags:
  - 读书笔记
---

```js
const p = new Promise((resovle, reject) => resolve('pass')).then(msg => console.log(msg)) // pass
```

## 创建一个Promise类

1. 它是一个构造方法，并且接收一个`function`作为参数，有`resolve`和`reject`两个方法。
2. 三种状态， 初始化为`pending`，可变为`fullfiled`或`rejected`。
3. then方法：
    
    - 接收`onFullfiled`,`onRejected`两个参数（可选），
    - 若两个参数不为函数，则忽略
    - `onFullfiled`接收promise中的`value`作为参数, 在`fullfiled`状态时被调用
    - `onRejected`接收promise中的`reason`作为参数，在`rejected`时被调用
    - then必须返回一个`promise`对象
    - then可以在同一promise中被多次调用

## 实现1

```js
const PENDING = 'pending';
const FULLFILED = 'fullfiled';
const REJECTED = 'rejected';
class MyPromise {
  constructor(callback) {
    this.state = PENDING;
    this.value = null;
    this.reason = null;
    callback(this.resolve, this.reject)
  }
  resolve = (value) => {
    this.value = value;
    this.state = FULLFILED;
  }
  reject = (reason) => {
    this.reason = reason;
    this.state = REJECTED;
  }
  then = (onFullfiled, onRejected) => {
    if (this.state === FULLFILED) {
      return new MyPromise((resolve, reject) => {
        try {
          const result = onFullfiled(this.value);
          // 如果onFullfiled返回promise对象，调用then方法
          if (result instanceof MyPromise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        } catch (err) {
          reject(err);
        }
      })
    }
    if (this.state === REJECTED) {
      return new MyPromise((resolve, reject) => {
        try {
          const result = onRejected(this.reason);
          if (result instanceof MyPromise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        } catch (err) {
          reject(err)
        }
      })
    }
  }
}

const p = new MyPromise((r) => r(2)).then(msg => new MyPromise(r => r(msg)).then(msg2 => console.log(msg2)))
// 2
```

至此，我们的`MyPromise`已经基本可以运行了，但存在一个很严重的缺陷，如果遇到异步请求时，`resolve`不能按上下文执行，导致then方法执行失败！

```js
/**
 * 定时任务，调用then方法时promise仍处于pending状态！此时不会返回任何东西
 */
const p2 = new MyPromise(r => setTimeout(() => {
  r(2)
}, 0)).then(msg => console.log(msg))
```

js已经把then方法执行了，但setTimeout中的方法还在`eventLoop`中等待执行

## 异步解决方法

- 将`then`中的方法保存起来，等待`resolve`或`reject`执行后再调用刚刚保存的`then`中的方法。

- 由于这个期间可能有多个`then`方法执行，所以需要用一个数据保存这些方法

- 关键改造
  - `then`方法中的判断条件由`fullfiled`/`rejected`变为`pending`
  - 增加两个数组记录`then`方法
  - `rejectArr`记录所有失败调用的函数： `reason => fn(reason)`, reason由触发`reject`方法时传入
  - `resolveArr`记录所有成功调用的函数：`value => fn(value)`,value由`resolve`方法触发

```js
class MyPromise {
  constructor(callback) {
    // ...
    this.rejectArr = [];
    this.resovleArr = [];
  }
  resolve = (value) => {
    if (this.state === PENDING) {
      this.state = FULLFILED;
      this.value = value;
      this.resolveArr.forEach(fn => fn(value))
    }
  }
  reject = (reason) => {
    if (this.state === PENDING) {
      this.state = REJECTED;
      this.reason = reason;
      this.rejectArr.forEach(fn => fn(reason))
    }
  }
  then  = (onFullfiled, onRejected) => {
    if (this.state === PENDING) {
      return new MyPromise((resolve, reject) => {
        try {
           this.resolveArr.push((value) => {
              // while resolved do
              const result = onFullfiled(value);
              if (result instanceof MyPromise) {
                // result返回promise对象，执行then方法
                result.then(resolve, reject);
              } else {
                resolve(result);
              }
           })
           this.rejectArr.push(reason => {
             // while rejected do
             const result = onRejected(reason);
             if (result instanceof MyPromise) {
               result.then(resolve, reject)
             } else {
               resolev(result)
             }
           })
        } catch(err) {
          reject(err)
        }
      })
    }
  }
}

```