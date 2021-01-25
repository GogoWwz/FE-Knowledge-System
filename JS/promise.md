## Promise

今天，我们来学习promise

怎么来学习呢？

个人觉得，对于Promise来说，就是手写一个，简单粗暴

话不多说，直接开始

### Promise A+规范

写promise之前呢，我们先来了解一下promise的规范，不然我们不知道要实现的玩意儿到底是要具备什么功能

Promise对象遵循的是[Promise A+规范 ](https://promisesaplus.com/)

大家可以去看链接，我这里就做一下摘要

#### 术语

- promise：一个具有符合规范方法的`then`方法的对象或者函数
- thenable：定义`then`方法的函数或对象
- value: 任何合法的javascript值
- exception：表示被`throw`语句抛出的值
- reason：表示promise为什么rejected（失败）的值

#### 状态

一个promise对象一定是处于 pending(等待中)、fulfilled(被处理)、rejected(被拒绝) 之中的一种状态

- 当处于pending状态的时候，promise可以转换为 fulfilled 状态 或者 rejected 状态
- 当处于 fulfilled 或者 rejected 状态的时候，promise对象不可能转换为其他状态
- 当处于 fulfilled 的时候，必须有个不可更改的 value
- 当处于 rejected 的时候，必须有个不可更改的 reason

特别要注意，三四两点中的 **不可更改** 的数据指的是同一个引用，并不是指引用内部不可变

#### then

promise对象必须提供一个then方法，用来访问promise当前或者最终状态时的 value 或者 reason

`then`方法接收两个参数：

```javascript
promise.then(onFulfilled, onRejected)
```

- `onFulfilled`、`onRejected`都是可选参数且必须是函数，如果不是函数，就忽略
- `onFulfilled`、`onRejected` 必须在promise实现之后才能调用，并将promise的 value 或者 reason 作为其第一个参数，且不能多次调用
- `then`函数可以被调用多次，`onFulfilled`、`onRejected` 必须按顺序执行
- 是一个微任务，等执行环境中的js代码执行完毕之后开始执行，先于event queue

#### 链式调用规则（重点）

这里才是我们实现Promise函数的难点，因为我们要遵循许多规则，我整理了一下，实际上就下面几点：

- `then`函数返回新的promise：也就是说链式调用我们不能使用`return this`的方式，因为每次的promise都是一个新的对象，这个很重要，涉及到我们实现链式调用的思路

  原因也很简单，promise规范的状态值只能由pending -> fulfilled或者rejected，不可逆，如果都是同一个无法实现。

  那规范为什么要这么定义呢？个人的理解是为了promise更可控，也就是immutable数据的重要性

  ```javascript
  let promise = new Promise((resolve, reject) => {
    resolve(1)
  })
  
  let promise1 = promise.then(() => {
    return 2
  })
  promise.then(res => {
    console.log(res) // 1
  })
  promise1.then((res) => {
    console.log(res) // 2
  })
  ```

  如果是返回同一个，那实际上打印的都是2，那中间态我们就无法控制了，这是我个人的理解

  而且这样状态流很清晰，返回新的我不需要管之前的流动，每个then都是pending -> xxxxx, 而返回同一个的话状态流就必须从第一个then开始管理了，何必呢？

- 上一个promise的 onFulfilled、onRejected 执行出错或者`throw`语句时，返回的promise被reject，处于rejected状态

- 上一个promise的 onFulfilled、onRejected 返回合法js值（undefined、promise对象等），需要定义`resolvePromise`函数对其进行处理，再决定下一个promise的状态

- 上一个promise没有处理函数的话，返回的promise与上一个promise的状态一直，value 或者 reason均相同

- 进入链式onFulfilled的执行条件：1、promise的value值为js任意合法值；2、promise的value是个被resolve的promise对象

- 进入链式onRejected的执行条件：1、错误或者throw语句；2、promise的value是个被reject的promise对象

### 基本版Promise

```javascript
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"

class _Promise {
  constructor(excutor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    const resolve = (value) => {
      if(this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
      }
    }

    const reject = (reason) => {
      if(this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
      }
    }
    try {
      excutor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }
  then(onFulfilled, onRejected) {
    if(this.status === FULFILLED) {
      onFulfilled(this.value)
    }
    if(this.status === REJECTED) {
      onRejected(this.reason)
    }
  }
}

let promise = new _Promise((resolve, reject) => {
  // resolve("fulfilled")
  // reject("rejected")
  throw new Error("exception")
})

promise.then((value) => {
  console.log("then value: ", value)
}, (reason) => {
  console.log("then reason: ", reason)
})
```

###  处理异步状态

上面的代码我们已经实现了一个最最基本的promise了，但是有个问题：

传入promise的构造器函数如果是异步的，就会存在当我们调用then的时候，此时的promise还是处在pending状态

所以，onFulfilled 或者 onRejected 都无法执行

那么我们怎么监听promise的 status，并且在其转为 fulfilled 或者 rejected 的时候去通知then函数去执行呢？

这个时候就可以使用发布订阅模式了，不清楚的可以看下我关于设计模式的文章

思路：当我们调用then的时候，如果此时的promise 处于pending状态， 我们就将 onFulfilled 或者 onRejected 函数加到promise的订阅组里，当promise被resolve 或者 reject的时候，通知订阅组进行执行

```javascript
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"

class _Promise {
  constructor(excutor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if(this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onFulfilledFns.forEach(fn => {
          fn(this.value)
        })
      }
    }

    const reject = (reason) => {
      if(this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedFns.forEach(fn => {
          fn(this.reason)
        })
      }
    }
    try {
      excutor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }
  then(onFulfilled, onRejected) {
    if(this.status === FULFILLED) {
      onFulfilled(this.value)
    }
    if(this.status === REJECTED) {
      onRejected(this.reason)
    }

    // 如果处于pending状态，则需要等待promise的状态
    if(this.status === PENDING) {
      this.onFulfilledFns.push(onFulfilled)
      this.onRejectedFns.push(onRejected)
    }
  }
}

let promise = new _Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("fulfilled")
  }, 2000)
  // resolve("fulfilled")
  // reject("rejected")
  // throw new Error("exception")
})

promise.then((value) => {
  console.log("then value: ", value)
}, (reason) => {
  console.log("then reason: ", reason)
})
```

### 链式调用完善

通过发布订阅模式，我们解决了单个promise对象的异步问题

现在就来实现链式调用，难倒是不难，重点是要遵循A+的规范:

```javascript

```







