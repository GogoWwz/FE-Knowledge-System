## this

this的问题，之前一直觉得自己就算不了解也算略知一二，应付平时的开发应用完全不在话下，后来，参加了字节的面试。

面试官很温柔的说：我看你简历上说很喜欢整理归纳总结知识点，那请你来说说你关于this的理解吧

我：好的，this是在函数被调用的时候绑定的.....然后会被bind、call、apply改变指向......然后箭头函数不会.....

面试官：那绑定有哪几种方式？

我：？？？

面试官：你知道this有优先级嘛？

我：？？？

面试官：所以你觉得是系统的理解过了吗

我：没......

真是个悲伤的故事，面试结束第一件事就是把简历调成自己看见，是真不好意思发简历了

并且，有了今天的这篇博客

### 为什么要使用this

先看一个例子：

```javascript
// 不使用this
function sayYou(context) {
    console.log(context.name)
}
let you = {
    name: "lss"
}
sayYou(you)

// 使用this
function sayMe() {
    console.log(this.name)
}
let me = {
    name: "wwz"
}
sayMe.call(me)
```

如果不使用this，我们是不是调用的时候都得显示传递一个context上下文对象来记录此时的调用者

上面代码中，只有传递一级还好，一旦多了，总不可能一级级把this传下去吧

**所以说，this提供了一种更优雅的方式，"隐式"传递了一个调用者对象**

### this绑定

从语义上理解，this应该指向的是这个对象本身，但JavaScript中的this真的是从语义上理解的这样吗？

```javascript
function fn(num) {
    console.log(num)
    this.count++
}

fn.count = 0

for(let i = 0; i < 5; i++) {
    fn(i)
}

// 0
// 1
// 2
// 3
// 4

console.log("fn.count：", fn.count) // 0
```

如果说this真的指向对象本身的话，循环里的打印说明fn确实被调用了5次，为什么fn的属性count还是0？

那会不会是指向当前的执行环境呢？

```javascript
function fn() {
    let a = 1
    bar()
}

function bar() {
    console.log(this.a)
}

fn() // undefined
```

很明显也不是，如果指向当前的执行环境那应该可以打印出 1 

这两种误解都排除之后，来看看真实的过程：

当一个函数被调用的时候，会创建一个活动记录（执行上下文），这个记录包含了函数的调用栈、调用方式、传参等信息，this就是这个记录的一个属性

关于this的绑定，我们只需要记住：**this是在函数被调用的时候绑定，指向绑定时候的执行环境，与在哪声明无关**