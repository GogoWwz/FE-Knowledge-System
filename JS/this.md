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

### this指向

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

说明this的指向语义地认为指向自身。

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

很明显也不是，bar调用时的环境栈为fn，如果是指向fn，那应该能打印出1

this指向的是调用时候的执行环境，这个毋庸置疑

那具体是怎么绑定的？遵循一个什么规则，下面就细说一下

### this的绑定方式

#### 默认绑定

独立函数调用的时候会遵循默认绑定的规则

什么是独立函数调用？

```javascript
function fn() {}
fn()
```

这就是个独立函数调用，函数被调用的时候没有任何的修饰符

大白话来讲，就是写了个函数直接简单粗暴就用了

这种情况下的this是绑定到哪里的呢？

```javascript
function fn1() {
    console.log(this)
    fn2()
}

function fn2() {
    console.log(this)
    fn3()
}

function fn3() {
    console.log(this)
}

fn1()
// Window
// Window
// Window
```

f1, f2, f3这三个函数，每个函数的调用位置都不同，context执行环境也不同，然后共同点是都是直接调用了

**所以，函数直接调用的情况下，this默认指向的都是window，严格模式下指向undefined**

#### 隐式绑定

书上的解释个人觉得确实有点晦涩，

白话一点，就是当函数有调用者的时候，就会触发隐式绑定的场景

看个最简单的例子：

```javascript
let obj = {
  name: "wwz",
  fn: fn
}

function fn() {
  console.log(this === obj)
  console.log(this.name)
}

obj.fn()
// true
// wwz
```

结果很明显，this指向了obj，js在这种场景下隐式地将this指向了obj

为什么？

**因为当一个函数被调用的时候，会创建一个活动记录（执行上下文），这个记录包含了函数的调用栈、调用方式、传参等信息，this指向的就是这个环境**

白话一点呢，就是谁调用了这个函数，this就指向谁

总结了之后好像是很简单，但是难就难在怎么找这个调用者

下面就用各种例子轰炸，加深理解，空想是会把自己绕进去的，例子之后都会放原因:

例子1：

```javascript
let obj = {
  name: "wwz",
  fn: function() {
    fn()
  }
}
function fn() {
  console.log(this.name)
}

obj.fn() // undefined

// 实际上fn的调用时直接调用的，所以此时是走默认绑定的规则，this指向window
// obj.fn()调用的是匿名函数，如果在这个函数里打印this.name，就可以打印出"wwz"了
```

例子2：

```javascript
let obj = {
  name: "wwz",
  fn: fn
}
let obj1 = {
  name: "lss",
  age: 20,
  obj: obj
}
function fn() {
  console.log(this.name)
  console.log(this.age)
}

obj1.obj.fn() // wwz undefined

// fn最后的调用者是obj，所以函数执行上下文绑定的也是obj
// 这里也看出来，函数的上下文并不像作用域链一样可以向上回溯，直到找到变量为止
```

个人感觉应该轰炸的差不多了吧

然后继续，隐式绑定会有一个问题：

**那就是在函数赋值或者作为参数传递之后，调用之后this会丢失，从而执行默认绑定**

还是先看例子：

```javascript
var name = "global"
let obj = {
  name: "wwz",
  fn: fn
}

function fn() {
  console.log(this) // Window
  console.log(this.name) // global
}

let bar = obj.fn

bar()
```

结果看出来，this丢失了，指向了window

其实用上面概括的也挺好理解：隐式绑定就是谁调用你指向谁

bar这函数没人调用啊，所以发生了默认绑定，指向了window

第二种情况就是传参的时候，this也会丢失：

```javascript
var name = "global"
let obj = {
  name: "wwz",
  fn: fn
}

function fn() {
  console.log(this) // Window
  console.log(this.name) // global
}

let bar = function(fn) {
  fn()
}

bar(obj.fn)
```

其实还是老样子，bar这函数没人调用啊，还是默认绑定

所以为什么react事件需要绑定this，因为react中事件是合成过的，我们的回调函数是作为参数传给了合成事件，不绑定this的话肯定就丢失了绑定到window上了

真要细说的话，就是：

赋值也好、传参也好，实际上都是用一个变量去指向这个函数，此时函数都没执行就别说上下文了。

其实，赋值、传参完全可以理解为"新声明"了一个函数，怎么绑定this调用者是谁就好了

#### 显式绑定

顾名思义就是开发者自己绑定this，涉及到的api实际上也就是`call`,`apply`

现在我们手写一个call来试试：

```javascript
var name = "global"
let obj = {
  name: "wwz",
  fn: fn
}

function fn(x, y) {
  console.log(this.name)  // wwz
  return x + y
}

Function.prototype._call = function(context, ...args) {
  let result
  // 用Symbol是为了防止和原属性名冲突
  let fn = Symbol("fn")
  // 之前说的，谁调用this就指向谁，_call此时被fn函数调用，所以此时this指向的就是fn的
  context[fn] = this
  result = context[fn](...args)
  delete context[fn]
}

let sum = obj.fn._call(obj, 1, 2) // 3
```

上面备注已经写的比较详细了，重点就是利用谁调用就指向谁的特性，将指向原函数的this赋值给context作为其属性，然后通过context去掉用过，这样就形成了this的指向更改了

但现在有个问题：关于this丢失的问题。我们怎么能拿到一个已经绑定好this的函数呢，call 和 apply都是直接调用。

我想在某个环境下保留一下this，然后在别的地方调用的时候还是保留的this指向，call 和 apply就不适合这种场景了

鉴于此，我们可以看看看下面例子：

```javascript
var name = "global"
let obj = {
  name: "wwz",
  fn: fn
}

function fn() {
  console.log(this) // obj
  console.log(this.name) // wwz
}

let bar = function() {
  fn.call(obj)
}

bar() 
```

这种绑定this的方式称为**硬绑定**，因为bar一旦声明了，内部的this就无法更改了，所以称为硬绑定。

利用硬绑定，配合call，这就是`bind`的实现原理：

```javascript
Function.prototype._bind = function(context, ...args) {
  let $fn = this
  return function() {
    let result = $fn.call(context, ...args)
    return result;
  }
}

let foo = obj.fn._bind(obj, 1, 3)
let sum1 = foo()
console.log(sum1)
```

个人认为当你能熟练的手写出`call`、`bind`原理时，隐式绑定和显示绑定的原理基本上已经掌握的七七八八了

#### new绑定

不可避免的，我们得先知道new做了什么：

- 创建一个新对象
- 新对象的[[proto]]属性指向构造函数的prototype
- 绑定this，将函数的this绑定到新对象上（实际上就是显示绑定了this）
- 返回新对象（如果构造函数没有返回对象的话）

然后我们来实现以下new

```javascript
function People() {
  console.log(this)
}

function _new(fn) {
  let result
  // 1、创建一个新对象
  let obj = {}
  // 2、__proto__属性指向构造函数原型
  obj.__proto__ = fn.prototype
  // 3、绑定this
  result = fn.call(obj)
  // 返回新对象
  return typeof result === 'object' ? result : obj
}

let person = _new(People)
```



### 总结

看完这个，再去刷this的题，刷个20来道，除开一些特别恶心的，闭着眼睛都能理清了