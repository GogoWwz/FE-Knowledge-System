## AJAX

### 创建xhr对象

由于现在已经基本不用兼容很老的ie，浏览器大多数也支持了`XMLHttpRequest`对象，所以直接创建即可：

```javascript
var xhr = new XMLHttpRequest()
```

### xhr.open

启动一个请求以备发送，也就是说**请求还未发送**

该方法接受三个参数：

- method: 请求的方法，`get`、`post`等
- url: 请求url
- async：是否为异步请求

```javascript
xhr.open("get", "test.php", true)
```

### xhr.send

发送请求

该方法接受一个参数：

- data：本次请求数据，如果没有，必须传`null`

```javascript
xhr.send(null)
```

### xhr.onreadystatechange

监听xhr对象的readyState变化，每次变化就会触发该函数

xhr对象的readyState属性有以下值：

- 0：请求尚未启动，即未调用`xhr.open`方法
- 1：已启动，但未发送，即未调用`xhr.send`方法
- 2：请求中，即请求已发送，但未收到相应
- 3：请求相应，已经开始接受部分数据
- 4：请求完成，数据接受完毕

因此，只有当readyState的值为4的时候，我们才算请求成功

同时，因为我们发送的是HTTP请求，本身也有一个`status`状态码，200时表示请求成功响应

所以我们最终只有在判断完上述的逻辑后，才算是一次完整成功的请求：

```javascript
xhr.onreadystatechange = function() {
    if(xhr.readyState === 4 && xhr.status === 200) {
        console.log("响应成功：", xhr)
    }
}
```

### xhr.setRequestHeader

对xhr对象发起的请求设置请求头

所以，该函数肯定是得在send前调用的

这里我们随便设置一个：

```javascript
xhr.setRequestHeader("MyHeader", "MyValue")
```

我们可以看下效果：

![image-20210310235202142](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210310235202142.png)

### 原生版 - GET

那么综上所述，我们就可以完成一个极简版本的原生的 GET 请求：

```javascript
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
	if(xhr.readyState === 4 && xhr.status === 200) {
        console.log("响应成功：", xhr)
    }
}

xhr.open("get", "xxx.php", true)
xhr.send(null)
```

### 原生版 - POST

有了上述的例子，相信封装一个POST请求并不难

如果我们不传参数，那么直接将open里的method参数改为 post 即可

但这样就没有意义了，所以我们一般是会将post请求模拟成表单的请求

```javascript
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
	if(xhr.readyState === 4 && xhr.status === 200) {
        console.log("响应成功：", xhr)
    }
}
xhr.open('post', 'xxx.php', true)
xhr.send()
```



