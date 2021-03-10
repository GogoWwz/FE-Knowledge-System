## render阶段

### 开始

render阶段的功能就是，对比最新的jsx 和 current树，来获得一颗 更新后的workInProgress树

因为reconciler阶段较多，所以在这里我将render 和 reconciler单独拆分为一个阶段

此处的render阶段指的是进入reconciler前的准备阶段

我们知道，react中的更新页面可以是 `ReactDom.render`、`setState`、`forceUpdate`、`useState`等

我们就来挨个看看

### ReactDom.render

一般来说，我们使用该方法都是在首次 mount 的时候

所以在首次mount的时候，该函数主要做了以下事情：

- 创建 FiberRootNode 节点，作为整个应用的根节点
- 创建 RootFiber 树，作为当前应用的节点树
- `FiberRootNode.current = RootFiber`、`RootFiber.stateNode = FiberRootNode`，FiberRootNode的current属性指向的树我们称之为 current树
- 此时由于还没进入render阶段，所以此时的current树实际上就是一个只包含 container信息的没有子节点的树

创建 FiberRootNode 执行栈：

`render` -> `legacyRenderSubtreeIntoContainer` -> `legacyCreateRootFromDOMContainer` -> `createLegacyRoot` -> `createRootImpl` -> `createContainer` -> `createFiberRoot`

创建 RootFiber 执行栈：

`render` -> `legacyRenderSubtreeIntoContainer` -> `legacyCreateRootFromDOMContainer` -> `createFiberRoot` -> `createHostRootFiber`

这样子执行完之后，目前拿到了 FiberRootNode节点 和 currentFiber树

然后回到`legacyRenderSubtreeIntoContainer`函数，上面我们已经有了初始的FiberRootNode节点 和 RootFier 树，

就会调用`unbatchedUpdates`去触发更新函数`updateContainer`，主要作用：

- 创建一个update对象，将更新的jsx元素作为payload

  ![image-20210306220649761](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210306220649761.png)

- `enqueueUpdate`：接受current fiber 和 上一步创建的 update 对象

  正常情况下，每个fiber节点都会有个初始的updateQueue属性，updateQueue.shared.pending存放着 update 对象

  如果上次的pending为空，说明是首次更新，

  ![image-20210306221131032](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210306221131032.png)

### 双缓存

我们可以发现，diff的过程实际上就是拿current树与jsx比对，生成workInProgress树的过程

那为什么要多加一棵workInProgress树呢？

看动画我们都知道动画是一帧帧放的，在播放下一帧前，先将前一帧清除

但如果说后一帧动画的绘制时间太长的话，从前一帧的清除到后一帧的显示之间间隔的时间就会产生白屏

这体验就不好了

所以，我们可以在内存中先画着下一帧的画面，等画好了之后直接用就好了，这样就不会有白屏

VDom渲染也是同样的道理，由于页面显示着我们的VDom树，如果内存中只有一颗VDom树的话，在将其生成新VDom树的计算过程时间太久的话，在新VDom出来前必然会有白屏。

所以，此时增加一棵workInProgress树用来表示更新好后的VDom树，这样直接替换current树，就不会产生白屏卡顿的体验了

