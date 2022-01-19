### 快速入门

[ 菜鸟教程)](https://www.runoob.com/typescript/ts-install.html)


### 配置

[tsconfig.json](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

```js
{  
  "compilerOptions": {  
    "target": "esnext",  
    "useDefineForClassFields": true,  
    "module": "esnext",  
    "moduleResolution": "node",  
    "strict": true,  
    "jsx": "preserve",  
    "sourceMap": true,  
    "resolveJsonModule": true,  
    "esModuleInterop": true,  
    "skipLibCheck": true,  
    "lib": ["esnext", "dom"],  
    "baseUrl": ".",  
    "paths": {  
      "@": ["src"],  
      "@/*": ["src/*"]  
    }  
  },  
  "include": ["*.d.ts","src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue","node_modules/@types"]  
}
```

- skipLibCheckd tsc忽略node_modules

```js
{
  "compilerOptions": {
    /* Basic Options */
    "target": "es5" /* target用于指定编译后js文件里的语法应该遵循哪个JavaScript的版本的版本目标: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */,
    "module": "commonjs" /* 用来指定编译后的js要使用的模块标准: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
    "lib": ["es6", "dom"] /* lib用于指定要包含在编译中的库文件 */,
    "allowJs": true,                       /* allowJs设置的值为true或false，用来指定是否允许编译js文件，默认是false，即不编译js文件 */
    "checkJs": true,                       /* checkJs的值为true或false，用来指定是否检查和报告js文件中的错误，默认是false */
    "jsx": "preserve",                     /* 指定jsx代码用于的开发环境: 'preserve', 'react-native', or 'react'. */
    "declaration": true,                   /* declaration的值为true或false，用来指定是否在编译的时候生成相应的".d.ts"声明文件。如果设为true，编译每个ts文件之后会生成一个js文件和一个声明文件。但是declaration和allowJs不能同时设为true */
    "declarationMap": true,                /* 值为true或false，指定是否为声明文件.d.ts生成map文件 */
    "sourceMap": true,                     /* sourceMap的值为true或false，用来指定编译时是否生成.map文件 */
    "outFile": "./",                       /* outFile用于指定将输出文件合并为一个文件，它的值为一个文件路径名。比如设置为"./dist/main.js"，则输出的文件为一个main.js文件。但是要注意，只有设置module的值为amd和system模块时才支持这个配置 */
    "outDir": "./",                        /* outDir用来指定输出文件夹，值为一个文件夹路径字符串，输出的文件都将放置在这个文件夹 */
    "rootDir": "./",                       /* 用来指定编译文件的根目录，编译器会在根目录查找入口文件，如果编译器发现以rootDir的值作为根目录查找入口文件并不会把所有文件加载进去的话会报错，但是不会停止编译 */
    "composite": true,                     /* 是否编译构建引用项目  */
    "incremental": true,                   /* Enable incremental compilation */
    "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
    "removeComments": true,                /* removeComments的值为true或false，用于指定是否将编译后的文件中的注释删掉，设为true的话即删掉注释，默认为false */
    "noEmit": true,                        /* 不生成编译文件，这个一般比较少用 */
    "importHelpers": true,                 /* importHelpers的值为true或false，指定是否引入tslib里的辅助工具函数，默认为false */
    "downlevelIteration": true,            /* 当target为'ES5' or 'ES3'时，为'for-of', spread, and destructuring'中的迭代器提供完全支持 */
    "isolatedModules": true,               /* isolatedModules的值为true或false，指定是否将每个文件作为单独的模块，默认为true，它不可以和declaration同时设定 */

    /* Strict Type-Checking Options */
    "strict": true /* strict的值为true或false，用于指定是否启动所有类型检查，如果设为true则会同时开启下面这几个严格类型检查，默认为false */,
    "noImplicitAny": true,                 /* noImplicitAny的值为true或false，如果我们没有为一些值设置明确的类型，编译器会默认认为这个值为any，如果noImplicitAny的值为true的话。则没有明确的类型会报错。默认值为false 

TS7006 false关闭检测
*/

    "strictNullChecks": true,              /* strictNullChecks为true时，null和undefined值不能赋给非这两种类型的值，别的类型也不能赋给他们，除了any类型。还有个例外就是undefined可以赋值给void类型 

TS2722 false关闭检测

*/
    "strictFunctionTypes": true,           /* strictFunctionTypes的值为true或false，用于指定是否使用函数参数双向协变检查 */
    "strictBindCallApply": true,           /* 设为true后会对bind、call和apply绑定的方法的参数的检测是严格检测的 */
    "strictPropertyInitialization": true,  /* 设为true后会检查类的非undefined属性是否已经在构造函数里初始化，如果要开启这项，需要同时开启strictNullChecks，默认为false */
   "noImplicitThis": true,                /* 当this表达式的值为any类型的时候，生成一个错误 */
    "alwaysStrict": true,                  /* alwaysStrict的值为true或false，指定始终以严格模式检查每个模块，并且在编译之后的js文件中加入"use strict"字符串，用来告诉浏览器该js为严格模式 */

    /* Additional Checks */
    "noUnusedLocals": true,                /* 用于检查是否有定义了但是没有使用的变量，对于这一点的检测，使用eslint可以在你书写代码的时候做提示，你可以配合使用。它的默认值为false */
    "noUnusedParameters": true,            /* 用于检查是否有在函数体中没有使用的参数，这个也可以配合eslint来做检查，默认为false */
    "noImplicitReturns": true,             /* 用于检查函数是否有返回值，设为true后，如果函数没有返回值则会提示，默认为false */
    "noFallthroughCasesInSwitch": true,    /* 用于检查switch中是否有case没有使用break跳出switch，默认为false */

    /* Module Resolution Options */
    "moduleResolution": "node",            /* 用于选择模块解析策略，有'node'和'classic'两种类型' */
    "baseUrl": "./",                       /* baseUrl用于设置解析非相对模块名称的基本目录，相对模块不会受baseUrl的影响 */
    "paths": {},                           /* 用于设置模块名称到基于baseUrl的路径映射 */
    "rootDirs": [],                        /* rootDirs可以指定一个路径列表，在构建时编译器会将这个路径列表中的路径的内容都放到一个文件夹中 */
    "typeRoots": [],                       /* typeRoots用来指定声明文件或文件夹的路径列表，如果指定了此项，则只有在这里列出的声明文件才会被加载 */
    "types": [],                           /* types用来指定需要包含的模块，只有在这里列出的模块的声明文件才会被加载进来 */
    "allowSyntheticDefaultImports": true,  /* 用来指定允许从没有默认导出的模块中默认导入 */
    "esModuleInterop": true /* 通过为导入内容创建命名空间，实现CommonJS和ES模块之间的互操作性 */,
    "preserveSymlinks": true,              /* 不把符号链接解析为其真实路径，具体可以了解下webpack和nodejs的symlink相关知识 */

    /* Source Map Options */
    "sourceRoot": "",                      /* sourceRoot用于指定调试器应该找到TypeScript文件而不是源文件位置，这个值会被写进.map文件里 */
    "mapRoot": "",                         /* mapRoot用于指定调试器找到映射文件而非生成文件的位置，指定map文件的根路径，该选项会影响.map文件中的sources属性 */
    "inlineSourceMap": true,               /* 指定是否将map文件的内容和js文件编译在同一个js文件中，如果设为true，则map的内容会以//# sourceMappingURL=然后拼接base64字符串的形式插入在js文件底部 */
    "inlineSources": true,                 /* 用于指定是否进一步将.ts文件的内容也包含到输入文件中 */

    /* Experimental Options */
    "experimentalDecorators": true /* 用于指定是否启用实验性的装饰器特性 */
    "emitDecoratorMetadata": true,         /* 用于指定是否为装饰器提供元数据支持，关于元数据，也是ES6的新标准，可以通过Reflect提供的静态方法获取元数据，如果需要使用Reflect的一些方法，需要引入ES2015.Reflect这个库 */
  }
  "files": [], // files可以配置一个数组列表，里面包含指定文件的相对或绝对路径，编译器在编译的时候只会编译包含在files中列出的文件，如果不指定，则取决于有没有设置include选项，如果没有include选项，则默认会编译根目录以及所有子目录中的文件。这里列出的路径必须是指定文件，而不是某个文件夹，而且不能使用* ? **/ 等通配符
  "include": [],  // include也可以指定要编译的路径列表，但是和files的区别在于，这里的路径可以是文件夹，也可以是文件，可以使用相对和绝对路径，而且可以使用通配符，比如"./src"即表示要编译src文件夹下的所有文件以及子文件夹的文件
  "exclude": [],  // exclude表示要排除的、不编译的文件，它也可以指定一个列表，规则和include一样，可以是文件或文件夹，可以是相对路径或绝对路径，可以使用通配符
  "extends": "",   // extends可以通过指定一个其他的tsconfig.json文件路径，来继承这个配置文件里的配置，继承来的文件的配置会覆盖当前文件定义的配置。TS在3.2版本开始，支持继承一个来自Node.js包的tsconfig.json配置文件
  "compileOnSave": true,  // compileOnSave的值是true或false，如果设为true，在我们编辑了项目中的文件保存的时候，编辑器会根据tsconfig.json中的配置重新生成文件，不过这个要编辑器支持
  "references": [],  // 一个对象数组，指定要引用的项目
}
```
### 类型断言

类型断言可以用来手动指定一个值的类型，即允许变量从一种类型更改为另一种类型。

当 S 类型是 T 类型的子集，或者 T 类型是 S 类型的子集时，S 能被成功断言成 T。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果你想这么做，你可以使用 any。

它之所以不被称为类型转换，是因为转换通常意味着某种运行时的支持。但是，类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。

```javascript
var str:string = '1'
var a = <number><any> str;
var b = str as any as Number
```

### 类型

当类型没有给出时，TypeScript 编译器利用类型推断来推断类型。

如果由于缺乏声明而不能推断出类型，那么它的类型被视作默认的动态 any 类型。

```javascript
var num = 2;    // 类型推断为 number
console.log("num 变量的值为 "+num); 
num = "12";    // 编译错误
console.log(num);
```

对于确实无法预知一个值的类型，应该使用`unknown`
```js
let a:unkown = 30;

a+1 //Error  无法直接操作，需要先校验类型

if(typeof a === 'number'){
	a+1;
}
```

类型支持交集和并集`|` `&` 

```js
type a = Cat|Dog
type a = Cat & Dog
```

#### 条件类型

```js
type IsString<T> = T extends string ? true : false;


type ToArray<T> = T extends unkonwn?T[]:T[]

type A = toArray<number> // number[]
type B = toArray<number|string> // number[]|string[]

// 内置条件类型

// Exclude<T,U> 在T不在U
type A = number | string;
type B = string;
type C = Exclude<A, B>; //number


//NonNullable<T>   排除unll和undefined
type A = number | null;
type B = NonNullable<A>


// ReturnType<F> 函数返回类型
type A =  ()=>number
type B = ReturnType<A> //number


```


#### literal类型

```js

type Style = 'none' | 'dotted' | 'dashed' | 'solid'; // truncated for brevity
type Color = `red` | 'green' | 'blue'; // truncated for brevity
 
type BorderStyle = `${number}px ${Style} ${Color}`;
 
let borderStyle: BorderStyle = '3px solid red';

```


根据字符串指定一个具体的类型

```js
type A = {
	num:number
	str:string
}

function on<Name extends Extract<keyof A, string>>(name:Name,para:A[Name]):void{
	
}


on('num',123)
on('str','123')


type B<Name extends Extract<keyof A, string>> = {

	name:Name,
	data:A[Name]
}

const b:B<'num'> = {
	name:'num',
	data:123
}
```


### 枚举

```js
enum Language{
	English,
	Chinese
}

// 枚举作为key时

let obj = {
	[Language.Chinese]:'ch'
}
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


带泛型的索引

```js
type shape = 'rect'|'ellipse'
interface item<Shape extends shape>{
	shape:Shape
}

type items<Shape extends shape> = {
	[index in Shape]:item<index>
}


const instance:items<shape> = {
	rect: {shape:'rect'}
	ellipse: {shape: 'ellipse'}
}
```
### keyof

该操作符可以用于获取某种类型的所有键，其返回类型是联合类型。

```js
interface Person {  
 name: string;  
 age: number;  
 location: string;  
}  
  
type K1 = keyof Person; // "name" | "age" | "location"  
type K2 = keyof Person[];  // number | "length" | "push" | "concat" | ...  
type K3 = keyof { [x: string]: Person };  // string | number
```

除了接口外，keyof 也可以用于操作类，比如：

```js
class Person {  
 name: string = "Semlinker";  
}  
  
let sname: keyof Person;  
sname = "name";  
```

若把 `sname = "name"` 改为 `sname = "age"` 的话，TypeScript 编译器会提示以下错误信息：

```js
Type '"age"' is not assignable to type '"name"'.
```


keyof 操作符除了支持接口和类之外，它也支持基本数据类型：


```js
let K1: keyof boolean; // let K1: "valueOf"  
let K2: keyof number; // let K2: "toString" | "toFixed" | "toExponential" | ...  
let K3: keyof symbol; // let K1: "valueOf"
```

此外 `keyof` 也称为输入索引类型查询，与之相对应的是索引访问类型，也称为查找类型。在语法上，它们看起来像属性或元素访问，但最终会被转换为类型：

```js
type P1 = Person["name"];  // string  
type P2 = Person["name" | "age"];  // string | number  
type P3 = string["charAt"];  // (pos: number) => string  
type P4 = string[]["push"];  // (...items: string[]) => number  
type P5 = string[][0];  // string
```



提取所有指定类型的key
```js
type A = {
	num:number
	str:string
}

type keys = Extract<keyof A,string>
```
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


> Type 'HTMLElement | null' is not assignable to type 'HTMLElement | undefined'. .ts(2322)

```js
// 在申明有null的变量后加 !
const portalDiv = document.getElementById('#your-element')!;
```


> Could not find a declaration file for module 'lodash'. '

需要在`tsconfig.jons`中加上配置`node_modules/@types`
```json
{
	"include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue","node_modules/@types"]
}
```

## 泛型

泛型声明的位置，有两种方式

```js
type Filter1 = {
	<T>(T): T;
};


let f1: Filter1 = <T>(arg:T)=>arg


type Filter2<T> = {
	(): T;
};
const f2: Filter2<number> = () => 1;

```
## 示例

### function的类型
```js
const func = function<T extends object>(paras:T){
}
```

复杂类型，可以如下定义
```js
const params:{name:string,age:number}
```

定义function的类型，可以通过`interface`构建

```js

const f1: () => void = ()=>console.log('f1');
f1()


interface Func{
    ():void
}

const f2:Func =  ()=>console.log('f2')
f2()
```


### 一个`仅仅导入/导出` 声明语法
```js
interface LiType {
	name: string;
}

export type { LiType as default };

// import type from  '...'
    
```


### this的类型

```js
function hello(this:Date){

}
```

### 强制转换为非null、undefined类型

```js
let a:string|undefined

let b:string = a!
```

### cast强转类型

```js
let a = '123'

...

(a as string).trim()
(<string>a).trim()
```

### 类似map的类型声明

```javascript
type MyMapLikeType = Record<string, IPerson>;
const peopleA: MyMapLikeType = {
    "a": { name: "joe" },
    "b": { name: "bart" },
};

enum Color{
	red,
	blue
}

type colorMap = Record<Color, string>;
// 枚举需要列出所有key
const peopleA: MyMapLikeType = {
	[Color.red]:'red',
	[Color.blue]:'blue'
};
```
