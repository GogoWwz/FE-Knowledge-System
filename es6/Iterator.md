## Itrator

### 开始

是一个非标准的函数，目前好像只有火狐浏览器实现了

在js中，表示集合概念的数据结构主要是：Array、Object、Map、Set

同时，我们还可以任意组合，比如Object的属性值是Array，Array中某一项是Map

我们就需要一种统一的方式来遍历这些集合类型的数据

Iterator就这这样一个函数，他的作用就是用来遍历这些数据结构

### 使用

`Iterator(obj, [keyOnly])`：

- obj: 可遍历对象
- keyOnly：调用next方法时，是否只返回key

iterator

```

```

