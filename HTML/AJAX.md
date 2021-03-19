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

但这样就没有意义了，所以我们一般是会将post请求模拟成表单的请求：

```javascript
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
	if(xhr.readyState === 4 && xhr.status === 200) {
        console.log("响应成功：", xhr)
    }
}

xhr.open('post', 'xxx.php', true)
// 设置请求头为表单提交时的contentType
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
xhr.send("username=wwz")
```

### AJAX 2.0

鉴于XHR已经被广泛使用，所以W3C也开始着手制定了相应的标准以及规范，于是就有了AJAX 2.0

在2.0中，ajax的功能更加强大了，当然，存在兼容性，所以大家了解一下即可

#### FormData

xhr对象可以直接发送FormData对象，且能自动识别`Content-Type`

```javascript
xhr.send(new FormData(form))
```

#### timeout

超时设置，当请求超过该值，就会触发ontimeout事件

```javascript
xhr.timeout = 1000 // 设置超时时间为1s
xhr.ontimeout = function() {
	console.log("请求超时:", xhr)
}
```

#### Progress Events

W3C的一个工作草案，定义了与服务器通信相关的事件：

- `loadstart`：在接收到响应数据的第一个字节时触发
- `progress`：在接收到响应期间不断触发
- `error`：在请求发生错误时触发
- `abort`：手动调用 `xhr.abort` 函数时触发
- `load`：在接收到完整响应数据时触发
- `loadend`：在通信完成或者触发error、abort、loader事件后触发

我们可以试试：

```javascript
xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
    	console.log("onreadystatechange: ", xhr)
    }
}
xhr.onloadstart = function() {
	console.log("onloadstart:", xhr)
}
xhr.onprogress = function() {
	console.log("onprogress:", xhr)
}
xhr.onload = function() {
	console.log("onload:", xhr)
}
xhr.onloadend = function() {
	console.log("onloadend:", xhr)
}
```

然后，对比着看看各个Progress对应的xhr状态，就可以有更深的理解了：

![image-20210312225055880](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210312225055880.png)

### Axios

axios实际上就是Promise版本的ajax，并且具备以下扩展功能：

- 支持Promise API
- 具备请求和响应请求全局拦截
- 支持防御CSRF