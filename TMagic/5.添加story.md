## 添加storybook

### storybook

文档：https://storybook.js.org/docs/react/get-started

版本：v6.1

开发环境：create-react-app

### 初始化

`npx sb init`：storybook会根据当前环境自动初始化一个最合适的本地环境

`npm run storybook`：启动服务

### 第一个story

初始化之后，项目根目录会出现一个`.storybook`的目录

其中`main.js`包含了一些基本配置：

![image-20201217220802772](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20201217220802772.png)

有两种引入的方式：js | ts | tsx | jsx 或者 mdx 的方式

这里我们就用tsx去写第一个story：

```typescript
import React from 'react';
import Button from './button';

export default {
  title: 'Tmagic/Button',
  component: Button
};

export const Default: React.VFC<{}> = () => <Button>Default</Button>;
```

这样引入还没有样式的效果，所以我们还需要在`.storybook/preview.js`中引入一下样式，这样就OK了：

![image-20201217234534978](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20201217234534978.png)

### 添加Actions

