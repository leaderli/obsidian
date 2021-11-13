## 直接运行ts文件

```shell
npm install -g ts-node
npm install @types/node  -g
```


`demo.ts`
```javascript
var message:string = "hello world"
console.log(message)
```

```shell
ts-node  demo.ts
```


## 问题



> Cannot find name 'console'

```shell
npm install -g ts-node
npm install @types/node  -g
```