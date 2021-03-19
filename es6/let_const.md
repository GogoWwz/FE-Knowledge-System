## let const

es6新引入的变量声明方式

用来解决`var`声明提升、可以重复声明、没有块级作用域的问题

存在**暂时型死区（temporal dead zone，TDZ）**：从当前作用域顶部到变量声明前，变量不可用

针对函数：

```javascript
console.log(b) // undefined 说明不会变量提升
{
  console.log(b) // function 提升到当前作用域
  let a = 1
  function b() {}
}
console.log(b) // function，也就是说，块级作用域对函数是不生效的，但不会变量提升
```

说明块级作用域中是可以声明函数的，且不影响外层作用域的访问，但是变量提升并不是提升到全局而是当前块级作用域

`var`同理

`const` 和 `let`差不多，不同点就是`const`变量是只读的

但注意，`const`的只读是保存的地址：

```javascript
const a = {}
a.b = 1
console.log(a) // { b: 1 } 说明const存放的是地址，并不是完全意义上的只读
```

如果想真正冻结一个对象，可以使用`Object.freeze`：

```javascript
const a = Object.freeze({})
a.b = 1
console.log(a) // {} 是真正的只读，严格模式下报错，普通模式下无效
```

还有一个值得注意的问题是

在es5的环境中，全局作用域与顶级环境对象的属性是挂钩的

顶级环境就是当前js的执行环境，浏览器中就是Windows对象，Node环境下就是global对象

在es6中，这两个概念是脱钩的，也就是说，全局是全局，顶级是顶级，`let` `const`声明的全局变量不会挂到顶级对象上：

```javascript
let a = 1
const b = 2
console.log(window.a) // undefined
console.log(window.b) // undefined
```

顶级对象可以通过：`globalThis`访问，用来统一各个环境下不同的顶级对象变量