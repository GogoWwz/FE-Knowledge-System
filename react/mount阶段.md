## mount阶段

该阶段主要做了以下事情：

- 创建 FiberRootNode 节点，作为整个应用的根节点
- 创建 RootFiber 树，作为当前应用的节点树
- `FiberRootNode.current = RootFiber`、`RootFiber.stateNode = FiberRootNode`，FiberRootNode的current指向的树我们称之为 current树
- 此时由于还没进入render阶段，所以此时的current树实际上就是一个只包含 container信息的没有子节点的树

创建 FiberRootNode 执行栈：

`render` -> `legacyRenderSubtreeIntoContainer` -> `legacyCreateRootFromDOMContainer` -> `createLegacyRoot` -> `createRootImpl` -> `createContainer` -> `createFiberRoot`

创建 RootFiber 执行栈：

`render` -> `legacyRenderSubtreeIntoContainer` -> `legacyCreateRootFromDOMContainer` -> `createFiberRoot` -> `createHostRootFiber`



这样子执行完之后，目前拿到了 FiberRootNode节点 和 currentFiber树