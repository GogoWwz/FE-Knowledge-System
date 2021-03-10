## commit阶段

### 开始

当reconciler阶段结束后，最新的workInProgress树也就生成了

此时开始尽心将workInProgress树渲染到dom上的步骤

也就是 commit 阶段

注意：

**一旦进入到commit阶段，更新就是不可中断的了，所以中断是发生在render阶段的**

还是先来看调用栈：

`commitRoot` -> `runWithPriority` -> `Scheduler_runWithPriority` -> `commitRootImpl`

最终实现commit阶段的是在 `commitRootImpl` 函数中

commit过程又分为三个阶段：

- `before mutation`

- `mutation`
- `layout`

下面我们就来逐个进行梳理

### before mutation阶段

每次fiber需要更新，react称之为生成一次mutation

该阶段的作用就是在mutation前，对不同type的组件进行不同的钩子调用

执行该阶段的函数为：`commitBeforeMutationEffects`

#### commitBeforeMutationEffects_begin

ensureCorrectReturnPointer

