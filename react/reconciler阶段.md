## reconciler阶段

### 开始

该阶段就是协调阶段

重点工作就是比对current fiber 和 最新的 jsx children数组，来生成一棵新的workInProgress fiber

diff 算法，就发生在这个阶段

reconciler阶段的核心实际上就是 ChildReconciler 函数

这是一个协调构造器

一开始的时候，react就会通过 ChildReconciler 生成两个协调器

一个是 mountChildFibers

一个是 reconcileChildFibers

场景我们之后会说

我们在render阶段的时候理过了调用栈，是到了 workLoopSync 这个时候开始循环进行diff

`workLoopSync` -> `performUnitOfWork` -> `beginWork$1` -> `beginWork`

在 beginWork 函数中，会判断当前current fiber 的类型，来执行不同的update函数

但是最终，进入diff的入口函数均为 `reconcileChildren` 函数

### reconcileChildren

![image-20210228192947271](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210228192947271.png)

我们上面说了两个协调器，就是再这个时候使用的

#### mountChildFibers

当组件是未被渲染的新组件时，执行 mountChildFibers，直接进行生成fiber节点然后挂载的流程，不进行diff算法进行优化

#### reconcileChildFibers

如果current树存在，说明需要进行diff算法来生成 workInProgress树

当然不管 mountChildFibers 还是 reconcileChildFibers，后续的流程也是差不多的

如果是单一节点，就走单一节点的渲染逻辑

如果是多节点，就走多节点渲染的逻辑

### 准备工作

看代码前，我们先准备下我们的调试环境

根组件：

```react
function FunctionalDemo() {
  const [num, setNum] = useState(0)
  return (
    <div className="functionalDemo" onClick={() => setNum(num + 1)}>
      {
        num % 2 ? b : a
      }
    </div>
  )
}
```

我们可以自定义a、b组件就可以实现各种情况下的更新了

### 单节点渲染

调用栈：

`reconcileSingleElement` -> `placeSingleChild`

`reconcileSingleElement` 会拿到父fiber、current fiber、最新的jsx

### reconcileSingleElement

#### key改变

调用`deleteChild`将当前的fiber节点添加到return fiber的`deletion`属性上，标记为需要删除的fiber

然后创建一个新的fiber，将return fiber 

#### key不变

### 多节点渲染

![image-20210228211920435](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210228211920435.png)

画红框部分就是进入多节点diff的逻辑，即判断jsx children数组是否为Array关键函数为：`reconcileChildrenArray`

### reconcileChildrenArray

首先，我们可以思考一下，我们对dom会有哪些操作？

总结来看，可以归结为以下几种操作：

- 节点更新
- 节点增删
- 节点位置调换

从频率上来说，更新的操作肯定是最为频繁的

所以 react 的 diff 算法，优先会去处理节点更新的情况，之后再去处理节点增删或者顺序调换的情况

所以总共有两轮循环

接下来我们就详细解析一下这个函数

首先，我们来看看该函数中所定义的变量：

```react
// 存储最终需要返回的workInProgress节点
var resultingFirstChild = null;
// 上一个最新的 fiber节点，因为fiber之间是通过sibling连接的，所以需要通过该变量保留一下上次生成的fiber节点
var previousNewFiber = null;
// current节点
var oldFiber = currentFirstChild;
// fiber节点被更新index后最新的jsx下标
var lastPlacedIndex = 0;
// 当前循环到的jsx节点的下标
var newIdx = 0;
// 当前fiber节点的sibling节点
var nextOldFiber = null;
```

#### 不传key

1、最普通的情况 ———— 节点没多没少，仅仅是更新了props，且没有绑定key:

```react
const a = (
  <ul>
	<li>1</li>
 	<li>2</li>
  </ul>
)
const b = (
  <ul>
	<li>3</li>
	<li>4</li>
  </ul>
)
```

第一次循环，因为前后的元素key值均为null，type也没变，所以react认为可复用，调用 `updateSlot` 函数后，返回一个利用老的fiber更新后的 `newFiber`

然后调用 `placeChild`，将对应jsx 元素的下标`newIndex` 赋值给 `newFiber` 节点的 `index`属性，用来正确表示 fiber节点在dom中的位置，同时返回最新的下标

然后，利用 `previousNewFiber` 变量，判断其是否存在来判断当前 `newFiber` 是否为child。

- 如果没有，说明就是 child，所以首个fiber就是：`resultingFirstChild = newFiber`

- 如果有了，说明是 sibling，所以后续通过sibling连接：`previousNewFiber.sibling = newFiber`

然后重置一下`previousNewFiber`、`oldFiber`之后，即可进行下一次的循环

循环完成

判断`newIdx`与jsx children的长度是否一致，上述情况下肯定是一样的

所以此时`resultingFirstChild `已经更新完毕了

调用 `deleteRemainingChildren` 一下多余节点（我们这种情况是是没啥必要的，因为没有减少fiber节点）

将其返回作为上一个workInProgress节点的child节点即可

2、节点更新，但更新了type：

```react
const a = (
  <ul>
	<li>1</li>
 	<li>2</li>
  </ul>
)
const b = (
  <ul>
	<div>3</div>
	<li>4</li>
  </ul>
)
```

第一次循环，因为前后的type改变，react认为不可复用，于是`updateSlot`后，是根据最新的jsx生成的`newFiber`

但此时current 树上还残留着老的fiber，所以这种情况需要将老的fiber放到current树的`deletion`中，标记删除，也就是这段逻辑：

![image-20210302000041155](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210302000041155.png)

后续逻辑就就和第一种情况相同了，这里我们先保留一个问题：我们需要返回的是`resultingFirstChild`，为什么还要将current树上将老fiber标记为deletion呢，反正用不到

3、节点增加到末尾：

```react
const a = (
  <ul>
	<li>1</li>
 	<li>2</li>
  </ul>
)
const b = (
  <ul>
	<li>1</li>
    <li>2</li>
    <li>3</li>
  </ul>
)
```

这种情况，肯定是 current fiber也就是`oldFiber`先为null，所以跳出了循环，`newIndex`为2，与jsx children的长度3不同，所以不会跟第一种情况一样返回

进入到后续的逻辑

判断`oldFiber`是否存在，因为`oldFiber`就是current fiber，没有current，说明是新增的jsx元素

于是从`newIndex`开始循环jsx children数组，分别生成新的fiber节点后，用第一种情况相同的逻辑，生成`resultingFirstChild`直接返回即可

4、节点增加到中间

这里我们可以发现，如果是相同type，且其他同级的元素的type也都相同的情况下，和第二种情况实际上就是一样的逻辑

但是，如果插入了不同的type的元素，逻辑就又不一样了，也就是说，插入元素后导致对应的fiber 和 jsx 的type对不上了的情况：

```react
const a = (
  <ul>
	<li>1</li>
 	<li>2</li>
  </ul>
)
const b = (
  <ul>
	<li>1</li>
  	<div>11</div>
    <li>2</li>
  </ul>
)
```

实际上就是上述第2中和第3种情况的组合而已，即从增加的那个jsx开始更新，更新完毕后对剩余的jsx进行添加

5、节点删除末尾

```react
const a = (
  <ul>
	<li>1</li>
 	<li>2</li>
    <li>3</li>
  </ul>
)
const b = (
  <ul>
	<li>1</li>
    <li>2</li>
  </ul>
)
```

同理，第一次循环结束之后，肯定是jsx children先遍历完，和第一种情况一样，会得到一棵`resultingFirstChild`

此时`newIndex`为jsx children 的长度，所以两者肯定相等

这个时候调用`deleteRemainingChildren`函数，将剩下的节点删除后，返回`resultingFirstChild`即可

6、节点删除中间

```react
const a = (
  <ul>
	<li>1</li>
 	<div>2</div>
    <li>3</li>
  </ul>
)
const b = (
  <ul>
	<li>1</li>
    <li>2</li>
  </ul>
)
```

和第四种情况类似，也是删减后的元素逐级更新，更新完jsx后，将多余的jsx删除

我们上面基本已经总结了没有key的所有情况，我们可以发现，不绑定key，不管如何都会创建一个新的fiber节点，不管是复用current 还是 创建新fiber，都是新建的过程

对于复用旧fiber创建新fiber、更新了type导致节点不可复用重建的fiber，这两种的性能开销是不可避免的

但是增删过程中会一种情况，就是我其他项都没变，只是多了一个新的，或者少了一个

如果仅仅是按照上面的流程，会都导致了一个问题：

就是后续的fiber虽然没改变是复用的，但因为同级对比，要么type没变，用旧的fiber生成一个fiber，要么type变了，生成了一个新fiber

不管哪种，都是浪费的性能开销，完全没必要

所以react文档在协调那一章建议我们加key，就是为了避免重复创建的性能开销

#### 传key

其实我们可以发现，上面不传key的时候，实际上就是key均为null的时候

前后的key都是相同的，所以我们只需要再讨论key不同的情况即可：

1、前后key均不同：

```react
const a = (
  <ul>
	<li key={1}>1</li>
 	<li key={2}>2</li>
  </ul>
)
const b = (
  <ul>
	<li key={3}>3</li>
	<div key={4}>4</div>
  </ul>
)
```

这个例子中我们既更新了props，也更新了type，因为走的逻辑是相同的

第一次循环的时候，因为key不同，react认为fiber不可复用，所以调用`updateSlot`后生成的新fiber为null，直接跳出循环，不走节点更新的流程了

个人理解，因为fiber是个链表，链表中的某一环已知是不可用了，后续的节点肯定也没必要继续去循环了

所以直接走到key更改的逻辑

先调用 `mapRemainingChildren`，该函数的作用就是利用return fiber和 current fiber，生成一个{key: fiber}的map对象

然后，从`newIndex`开始，循环 jsx children数组

这个时候调用 `updateFromMap`，判断对应jsx key 是否存在在hash表中，如果有，直接用之前的fiber就好了，如果没有，返回null

后续再通过`updateElement`去更新fiber即可

后续的逻辑实际上就和之前的循环挂载差不多了，生成`resultingFirstChild`返回即可

在该用例中，key不存在复用的情况，所以均是新生成的fiber之后再去挂载

2、key存在相同的情况：

```react
const a = (
  <ul>
	<li key={1}>1</li>
 	<li key={2}>2</li>
  </ul>
)
const b = (
  <ul>
	<li key={1}>1</li>
	<div key={3}>3</div>
	<li key={2}>2</li>
  </ul>
)
```

这就是我们说过的，存在可复用节点的情况，就是key为2的li，我们是没必要进行重新创建的

所以当第一轮循环更新完key为1的li后，循环到div时，key值发生了改变，`newIndex`为1

直接进到我们上述的情况，生成了一个key为2的`existingChildren` map表（因为current已经指向了 key为1 的li的 sibling了，所以只有key为2的li）

然后从1开始循环jsx children，新创建了一个div的fiber，然后通过hash表的查找，直接用了表中的fiber

后续与之前一致

我们可以发现，如果没有传key，key为2的li是不是又要重新创建一次？

哈希表查找的时间复杂度是O(1)，相对创建一个fiber的开销便宜太多了

### 总结

这也就是react diff过程中为什么需要传key的意义：

**为了不重复创建可复用的节点的fiber节点，来提升性能**

另外，react官网中还说了一种就是当元素type改变的时候，会直接重新创建新的fiber节点，实际上就是我们不传key中的第二种情况

此外，由于diff的过程在react中是循环链表，将复杂度从O(n^3)直接降到了O(n)，循环肯定是同级比较
所以即使不同级之间是相同key，react也是识别不到的，所以我们平时尽量少做跨层级的操作

总结起来，react diff的优化有以下三点：

1、利用链表循环，将原本O(n^3)的复杂度降为了O(n)

2、type如果发生了变化，直接创建新的fiber节点，直接进行重建流程

3、利用key，实现对同级同key节点的复用，避免重建开销



最后我们总结一下更新时候的render的大致流程：

1、复用current fiber生成一颗初始的 workInProgress fiber

2、通过递归workInProgress fiber，以及diff算法，对workInProgress进行更新，获得一棵 reconcile 之后的最新 workInProgress fiebr

3、将 workInProgress fiber 赋值给root 的 `finishedWork` 属性

4、reconciler阶段完成，进入commit阶段：`commitRoot`







