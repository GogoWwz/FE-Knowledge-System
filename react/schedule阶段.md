## schedule阶段

### 开始

调度阶段，在该阶段，react实现了两个功能：

- 时间切片（timing splicing）
- 优先级调度

为什么要增加这么一个阶段呢？

我们可以用一段demo来看看，下面的例子是渲染3000个li：

```react
ReactDOM.render(
  <div className="App">
    <ul>
      { 
        Array(3000).fill(0).map((_, i) => (<li key={i} >{i}</li>))
      }
    </ul>
  </div>,
  document.getElementById('root')
);
```

我们知道，legacy模式下的render方法是同步的，所以当我们执行这一整套流程的时候，执行栈时这样的：

![image-20210305000048644](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210305000048644.png)

可以看到，一口气执行不带停的，这段时间执行170ms

我们知道浏览器的fps一般是60，也就是说每16ms左右会刷新一次屏幕

我们也知道浏览器是单线程的，因此这个任务完全阻塞了浏览器的渲染

此时是无法进行其他操作的，页面也就出现了卡顿

但是，当我们用concurrent模式后，就不一样了：

```
ReactDOM.createRoot(rootNode).render(<App />)
```

下面是开启concurrent模式后的task执行方式:

![image-20210305001435105](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210305001435105.png)

很明显的看到整个任务变成了中断式的了，这就是concurrent模式下的异步渲染

每个task的事件都在5ms左右，最后一个因为要连带着commit阶段，所以时间是9ms

