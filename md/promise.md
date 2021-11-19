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

Promise实例生成后，可用then方法指定两种状态回调函数。then方法可以接受两个回调函数作为参数，返回一个新的Promise实例。

`Promise.prototype.then(onFulfilled, onRejected);`

1. onFulfilled 当前实例变成fulfilled状态时，该参数作为回调函数被调用。
2. onRejected 当前实例变成reject状态时，该参数作为回调函数被调用。



```js
function sleep(ms) {
    return new Promise(function(resolve, reject) {
        setTimeout(resolve, ms);
    })
}
sleep(500).then( ()=> console.log("finished"),()=> console.log("failed"));
```


其执行顺序

```js
let promise = new Promise(function(resolve, reject){
    console.log("AAA");
    resolve()
});
promise.then(() => console.log("BBB"));
console.log("CCC")

// AAA
// CCC
// BBB
```


链式调用，每个then都会返回一个promise实例，如果没有显式的返回promise实例，就会默认返回一个调用resolve()的Promise实例

```js
let promise = new Promise(function (resolve, reject) {
    console.log("A");
    resolve()
});
promise
.then(() => console.log("B"))
.then(() => { 
    new Promise(resolve=>{
        resolve()
    }).then(()=>{
        console.log('inner ');
    })
    console.log("C"); 
});
console.log("G")

//A
//G
//B
//C
//inner 
//C
//reject
```
