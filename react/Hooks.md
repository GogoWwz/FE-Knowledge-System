## Hooks

我们从源码来剖析一下hooks的原理

### 入口

`renderWithHooks` -> `resolveDispatcher` 我们可以先看一张截图：

![image-20210315220649607](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210315220649607.png)

还有很多hooks没有截，我们看到，不管哪个hooks，流程均是通过`resolveDispatcher`函数来生成一个`dispatcher`后，再返回对应的hooks

### resolveDispatcher

那这个函数我们肯定得分析一下：