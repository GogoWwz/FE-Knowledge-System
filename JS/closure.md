##  closure

闭包这玩意儿，反正在我这儿属于，看完不过如此，没过多久模模糊糊

后来想了想，是因为自己从来没试过去玩闭包，也就是单方面的去学习书本、博客输送给我的知识

也就造成理解不深的问题

所以，今天打算换个思路，自己给自己出题，自己玩，然后说出自己的理解

首先，先看个例子：

```javascript
// 每隔一秒打印i
for(var i = 0; i < 5; i++) {
  setTimeout(() => {
    console.log(i)
  }, i * 1000)
}
// 5
// 5
// 5
// 5
// 5
```

本来想每隔一秒当时的i，因为定时器是宏任务，所以等for循环执行完了，才执行定时器，所以打印了五个5

我想了想，先不管闭包，我们尝试实现我们的需求

```javascript
// 利用bind保留当前i
for(var i = 0; i < 5; i++) {
  setTimeout((function(j) {
    console.log(j)
  }).bind(null, i), i * 1000)
}

// 利用立即执行函数中的作用域保留i
for(var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(() => {
      console.log(j)
    }, j * 1000)
  })(i)
}

// 利用let自带的块及级用域
for(let i = 0; i < 5; i++) {
  setTimeout(() => {
    console.log(i)
  }, i * 1000)
}
```

个人只能想到上面集中情况

