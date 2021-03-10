## HTML5 API

这里我们来整理一下常用的HTML5的 API

### geolocation - 地理位置

#### useragent.geolocation.getCurrentPosition

```javascript
navigator.geolocation.getCurrentPosition(pos => {
    console.log(pos)
})
```

该函数会返回一个地理位置信息 GeolocationPosition 对象：

![image-20210310222233816](C:\Users\wwz\AppData\Roaming\Typora\typora-user-images\image-20210310222233816.png)

#### useragent.geolocation.watchPosition

```javascript
navigator.geolocation.watchPosition(pos => {
    console.log(pos)
})
```

该函数会监听用户的地理位置，如果有变动，则会触发回调函数

#### useragent.geolocation.clearWatch

```javascript
navigator.geolocation.clearWatch(pos => {
    console.log(pos)
})
```

取消对用户位置的监听