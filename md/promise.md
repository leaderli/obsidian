#js 
Promise是异步编程的一种解决方案。有三种状态

1. `Pending`
2. `Fuilfilled`
3. `Rejected`


```js
var promise = new Promise(function(resolve, reject){
    // ... some code
    
    if (/* 异步操作成功 */) {
        resolve(value);
    } else {
        reject(error);
    }
})
```


promise构造函数接收一个函数作为参数，该函数的两个参数分别为resolve和reject，它们由JavaScript引擎提供。

- resolve函数将`Pending -> Fulfilled`
- reject函数将`Pending -> Rejected`

Promise实例生成后，可用then方法指定两种状态回调函数。
