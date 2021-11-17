## 快速入门

[ 菜鸟教程)](https://www.runoob.com/typescript/ts-install.html)


一些语法

### 类型断言

类型断言可以用来手动指定一个值的类型，即允许变量从一种类型更改为另一种类型。

当 S 类型是 T 类型的子集，或者 T 类型是 S 类型的子集时，S 能被成功断言成 T。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果你想这么做，你可以使用 any。

它之所以不被称为类型转换，是因为转换通常意味着某种运行时的支持。但是，类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。

```javascript
var str:string = '1'
var a = <number><any> str;
var b = str as any as Number
```

### 类型推断

当类型没有给出时，TypeScript 编译器利用类型推断来推断类型。

如果由于缺乏声明而不能推断出类型，那么它的类型被视作默认的动态 any 类型。

```javascript
var num = 2;    // 类型推断为 number
console.log("num 变量的值为 "+num); 
num = "12";    // 编译错误
console.log(num);
```

### 可索引的类型
与使用接口描述函数类型差不多，我们也可以描述那些能够“通过索引得到”的类型，比如`a[10]`或`ageMap["daniel"]`。 可索引类型具有一个 `索引签名`，它描述了`对象索引的类型`，还有相应的`索引返回值类型`。
```js
interface StringArray{
	[index: number]: string
}
let arr:StringArray = ["1","2"]

let a1:String = arr[0]
```

上面例子里，我们定义了`StringArray`接口，它具有索引签名。 这个索引签名表示了当用 `number`去索引`StringArray`时会得到`string`类型的返回值。
### 变量作用域

```javascript
var global_num = 12          // 全局变量
class Numbers { 
   num_val = 13;             // 实例变量
   static sval = 10;         // 静态变量
   
   storeNum():void { 
      var local_num = 14;    // 局部变量
   } 
} 
console.log("全局变量为: "+global_num)  
console.log(Numbers.sval)   // 静态变量
var obj = new Numbers(); 
console.log("实例变量: "+obj.num_val)
```

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