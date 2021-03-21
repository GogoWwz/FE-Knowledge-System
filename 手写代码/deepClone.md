## deepClone

深拷贝算是一个非常综合的代码，一个深拷贝涉及到递归、类型判断、循环引用、Symbol这些知识点

### 类型判断

一般情况下，我们判断类型都是用`typeof`操作符，但是`Arraty`、`null`均为object，一般情况是可以用`Array.isArray`来判断是否为数组

| 类型                                                 | 返回值                       |
| ---------------------------------------------------- | ---------------------------- |
| Object.prototype.toString.call(1)                    | "[object Number]"            |
| Object.prototype.toString.call("")                   | "[object String]"            |
| Object.prototype.toString.call(true)                 | "[object Boolean]"           |
| Object.prototype.toString.call(null)                 | "[object Null]"              |
| Object.prototype.toString.call(undefined)            | "[object Undefined]"         |
| Object.prototype.toString.call(() => {})             | "[object Function]"          |
| Object.prototype.toString.call([])                   | "[object Array]"             |
| Object.prototype.toString.call(Symbol())             | "[object Symbol]"            |
| Object.prototype.toString.call((async () => {}))     | "[object AsyncFunction]"     |
| Object.prototype.toString.call((function *fun() {})) | "[object GeneratorFunction]" |

用toString来判断，则比较靠谱

为什么要加call调用？

因为很多类型重写了`toString`方法，比如String类型的toString就是打印本身

Array的toString就是打印`array.join(",")`

### Symbol

当递归属性的时候，一般用到的就是`for in`或者`Object.key()`

这两个方式都只无法获取到`Symbol`属性的值

我们可以使用`Object.getOwnPropertySymbols`

或者直接使用es6的`Reflect.ownKeys`

这个方法可以返回所有的属性值

### 循环引用

利用Map对象，存放对象的地址和拷贝值，这样如果存在循环引用的地址的话，直接使用这个map中的对象即可

### 实现

```javascript
// 深拷贝
const testA = {
  str: "string",
  num: 1,
  [Symbol("symbol")]: "Symbol",
  arr: [1, 2, 4],
  obj: {
    child: {
      name: "child",
      subChild: {
        name: "subChild"
      }
    }
  },
  fn: function() {}
}
// 循环引用
testA.circle = testA

function deepClone(obj, map = new Map()) {
  let newObj = {}
  // 解决循环引用
  if(map.get(obj)) {
    return map.get(obj)
  }
  map.set(obj, newObj)

  // 递归属性
  Reflect.ownKeys(obj).forEach(key => {
    // 如果是不为null，不为Array的对象,递归处理
    if(Object.prototype.toString.call(obj[key]) === "[object Object]") {
      newObj[key] = deepClone(obj[key], map)
    } else {
      newObj[key] = obj[key]
    }
  })

  return newObj
}

const o = deepClone(testA)
console.log(o)
console.log(o === testA)
console.log(o.obj === testA.obj)
```



