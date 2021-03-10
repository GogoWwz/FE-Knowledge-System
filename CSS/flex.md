## flex

### 定义

Flexbox是flexible box的简称（注：意思是“灵活的盒子容器”），是CSS3引入的新的布局模式

它决定了元素如何在页面上排列，使它们能在不同的屏幕尺寸和设备下可预测地展现出来

它之所以被称为flexbox，是因为它能够扩展和收缩 flex 容器内的元素，以最大限度地填充可用空间

外层称为容器，

内部称为元素

### 容器属性

#### flex-direction

布局的主轴方向，即横着排还是竖着排

- `row`：主轴的方向为横向，顺序从左往右
- `row-reverse`：主轴方向为横向，顺序从右往左
- `columns`：主轴的方向为纵向，顺序从上到下
- `columns-reverse`：主轴方向为纵向，顺序从下到上

在flex中，有两根轴线，主轴上面说了，还有一根是交叉轴，交叉轴垂直于主轴

#### flex-wrap

元素是否可以换行

- `wrap`：可以换行
- `nowarp`：不换行，元素大小会被压缩
- `wrap-reverse`：可以换行，每行的顺序在交叉轴方向上相反

#### justify-content

元素在主轴上的对齐方式

- `flex-start`：元素以主轴开始处对齐
- `flex-end`：元素以主轴结尾处对齐
- `center`：元素在主轴上居中对齐
- `space-between`：元素在主轴上均匀分布，两侧元素贴合容器
- `space-around`：元素在主轴上均匀分布，两侧元素不贴合容器

#### align-items

元素在交叉轴上的对齐方式

- `flex-start`：元素以交叉轴开始处对齐
- `flex-end`：元素以交叉轴结束处对齐
- `center`：元素在交叉轴居中对齐

### 元素属性

#### order

定义元素的排列顺序，数值越小，排列越靠前

为整数，默认为0，可以为负数

#### flex-grow

定义元素的放大比例，即如果容器还有剩余空间，那么元素就会放大，默认为0不放大

![image-20210309223829702](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309223829702.png)

可以看到，都设置为0的时候，不放大

当我们都设置为1（其实任意相同值也行）的时候：

![image-20210309223946329](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309223946329.png)

可以看到每个元素就是等分，当我们把第一个设置为2后，可以发现：

![image-20210309224209645](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309224209645.png)

相应的第一个元素的宽度更大，所以元素的实际宽度和该属性的关系即为：

**元素宽度 = 剩余宽度 * (元素flex-grow / 所有flex-grow) + 元素原本宽度**

所以，利用该属性，我们很容易就能实现**定宽 + 自适应**的布局

如果是两列，定宽的元素`flex-grow:0;`，自适应的元素`flex-shrink: 1;`即可

如果是三列，左中右或上中下，那么两侧元素`flex-grow:0;`，中间元素`flex-shrink: 1;`即可

但是注意，当元素的总和超过总宽度，那么这个属性就会失效了，可以理解为上述的剩余宽度为0或负数

#### flex-shrink

与`flex-grow`相反，定义的就是元素的缩放比例，即如果元素宽度总和超过容器，那么缩放就会放大，默认为1

![image-20210309225921907](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309225921907.png)

可以看这张图，元素宽度虽然都设置了300px，但是实际由于缩放了之后，都变成了190px，

这个计算比例与`flex-grow`的计算比例类似：

**元素宽度 = 元素宽度 - 超出宽度 * ( 元素flex-shrink / flex-shrink总和 )**

通过这个公式，我们也很容易看出，当一个元素的`flex-shrink`为0时，即代表该元素不缩放

利用这个性质我们同样也能设置定宽 + 自适应的布局

如果是左右，设置定宽元素为0，自适应元素为1

如果是左中右或上中下，设置两侧为0，中间元素为1

可以发现，不管是`flex-grow`还是`flex-shrink`，当为0的时候，均代表不缩放，也就能让我们实现定宽+自适应的布局了

#### flex-basis

元素的本来大小，实际上就是元素的宽度

当我们使用该属性后，会优先使用该属性，而不是 width 或者 height 属性

![image-20210309231605616](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309231605616.png)

#### flex

`flex-grow`, `flex-shrink` 和 `flex-basis`的简写简写样式，即：

`flex: flex-grow flex-shrink? flex-basis?`

后面两个属性可选，默认为：0 1 auto

该属性还有两个快捷值：

`auto`: 0 1 auto

`none`: 0 0 auto

#### align-self

定义元素自身的对齐方式

比如容器上设置的对齐方式为 `center`

我们给第三个元素设置`align-self: flex-end;`

![image-20210309232625116](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210309232625116.png)