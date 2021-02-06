## schedule阶段

该阶段发生在生成新的workInProgress树之前

不管是`render`、`setState`、`forceUpdate`、hooks的`dispatchAction` 都会经过该步骤

`scheduleUpdateOnFiber`是任务调度阶段的起点

### 执行栈

如果是`render`函数：

`updateContainer` -> `scheduleUpdateOnFiber` -> `performSyncWorkOnRoot` -> `renderRootSync`



