### 后台运行库
pm2 是一个进程管理工具,可以用它来管理你的 node 进程，并查看 node 进程的状态，当然也支持性能监控，进程守护，负载均衡等功能

```shell
# 全局安装
npm install -g pm2
# 启动进程/应用
pm2 start app.js
# 重命名进程/应用
pm2 start app.js --name wb123
# 添加进程/应用 watch
pm2 start bin/www --watch
# 结束进程/应用
pm2 stop www
# 结束所有进程/应用
pm2 stop all
# 删除进程/应用
pm2 delete www
# 删除所有进程/应用
pm2 delete all
# 列出所有进程/应用
pm2 list
# 查看某个进程/应用具体情况
pm2 describe www
#  查看进程/应用的资源消耗情况
pm2 monit
# 查看pm2的日志
pm2 logs
# 若要查看某个进程/应用的日志,使用
pm2 logs www
# 重新启动进程/应用
pm2 restart www
# 重新启动所有进程/应用
pm2 restart all
```


### 镜像源管理工具

```shell
$ npm install -g nrm

# *表示当前正在使用的源
$ nrm ls
* npm -------- https://registry.npmjs.org/
  yarn ------- https://registry.yarnpkg.com/
  cnpm ------- http://r.cnpmjs.org/
  taobao ----- https://registry.npm.taobao.org/
  nj --------- https://registry.nodejitsu.com/
  npmMirror -- https://skimdb.npmjs.com/registry/
  edunpm ----- http://registry.enpmjs.org/

# 新增源
$ nrm add verdaccio http://centos7:4873
# 切换源
$ nrm use verdaccio
```

测试源响应速度

```shell
$ nrm test taobao
  taobao - 407ms
# 测试所有
nrm test
  npm ---- 336ms
  yarn --- 334ms
  cnpm --- 810ms
  taobao - 545ms
  nj ----- Fetch Error
  npmMirror  794ms
  edunpm - Fetch Error
* verdaccio  41ms

```

### 搭建私人仓库

1. 安装 verdaccio，使用 npm 全局安装即可。

   ```shell
   npm install –global verdaccio
   ```

2. 安装完成后，直接输入 verdaccio 命令即可运行

   ```shell
       $ verdaccio
       warn --- config file  - /home/li/.config/verdaccio/config.yaml
       warn --- Verdaccio started
       warn --- Plugin successfully loaded: verdaccio-htpasswd
       warn --- Plugin successfully loaded: verdaccio-audit
       warn --- http address - http://localhost:4873/ - verdaccio/4.7.2
   ```

   `config.yaml`是 verdaccio 的默认配置文件，为了能让外部访问，我们在其中添加

   ```yml
   listen: 0.0.0.0:4873
   ```

   我们使用 pm2 后台启动

   ```shell
   pm2 start verdaccio
   ```

3. 在自定义模块中，发布应用

   ```shell
    # 链接私有仓库
    nrm add verdaccio http://centos7:4873
    # 切换源
    nrm use verdaccio
    # 注册用户
    npm adduser
    # 发布
    npm publish
    # 下载我们发布的应用
    npm install test

   ```

   发布不成功，尝试使用最简格式`package.json`

   例如
   ![node模块_私有仓库.png](node模块_私有仓库.png)

4. verdaccio 存储 nodejs 包的地址
   `~/.local/share/verdaccio/storage`

### http

http post 请求

默认情况下请求报文格式为 json`Content-type: application/json`

```js
var request = require('request')
request(
  {
    url: url,
    method: "POST",
    json: requestData,
  },
  function (error, response, body) {
     ...
  }
);
```

### fs

读取文件为 base64

```js
const fs = require("fs");

let buff = fs.readFileSync("stack-abuse-logo.png");
let base64data = buff.toString("base64");
```

### moment

时间格式化模块

```shell
npm install -S -D moment
```

```javascript
import moment form 'moment'

moment().format('YYYY-MM-DD HH:mm:ss')
```

### [js-beautify](https://github.com/beautify-web/js-beautify)

格式化 js、html、css 代码片段用的插件

### express

拦截`/`请求，并打印请求报文

```shell
npm install --save express
npm install --save body-parser
```

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const app = express();
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());
app.post("/", (req, res) => {
  console.log(req.body);
  res.send("ok");
});
app.listen(5000, () => {
  console.log("start server at 5000");
});
```

### child_process

调用 shell 命令

```shell
npm install --save child_process
```

exec 的回调函数在命令执行后才会返回。

```javascript
const { exec } = require("child_process");
exec("cat *.js missing_file | wc -l", (error, stdout, stderr) => {
  if (error) {
    console.error(`执行出错: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  console.log(`stderr: ${stderr}`);
});
```

我们也可以通过`on`监听 shell 命令的管道来实时输出返回结果

```javascript
const { exec } = require("child_process");
let tail = exec("tail -f 1.log");
//data为byte数组
tail.stdout.on("data", (data) => {
  console.log(`${data}`);
});
tail.stderr.on("data", (data) => {
  console.log(`${data}`);
});

tail.on("close", (code) => {
  console.log(`子进程退出码：${code}`);
});
```


### xml2js

转换xml为js的工具类

```javascript
var xml2js = require('xml2js');
var xml = "<config><test>Hello</test><data>SomeData</data></config>";

var extractedData = "";
var parser = new xml2js.Parser();
parser.parseString(xml, function(err,result){
  //Extract the value from the data element
  extractedData = result['config']['data'];
  console.log(extractedData);
});
console.log("Note that you can't use value here if parseString is async; extractedData=", extractedData);
```

### lodash

![[lodash]]



###  dropzone

上传文件

```shell
npm i @types/dropzone -s
```


###  clipboard
复制粘贴
```shell
npm i ts-clipboard -s
```


### codemirror

代码编辑

```shell
npm i @types/codemirror -s
```


###  file-saver
保存为文件

```shell
npm i file-saver -s
npm i @types/file-saver -d

```


###  fuse

模糊搜索 [Live Demo | Fuse.js](https://fusejs.io/demo.html)

```shell
npm install --save fuse.js
```

fusejs搜索的list不支持响应式，list数据变化时需要重新new


```
const arr = [
    {
        title: 'Old Man\'s War',
        author: {
            firstName: 'John',
            lastName: 'Scalzi'
        }
    },
    {
        title: 'The Lock Artist',
        author: {
            firstName: 'Steve',
            lastName: 'Hamilton'
        }
    }
]
const options = {
    // isCaseSensitive: false,
    // includeScore: false,
    // shouldSort: true,
    // includeMatches: false,
    // findAllMatches: false,
    // minMatchCharLength: 1,
    // location: 0,
    // threshold: 0.6,
    // distance: 100,
    // useExtendedSearch: false,
    // ignoreLocation: false,
    // ignoreFieldNorm: false,
    keys: [
        'title',
        'author.firstName'
    ]
};

const fuse = new Fuse(myList);
const pattern = 'a';
console.log(fuse.search(pattern as string));
```

### jszip

```shell
npm install jszip -s
```


### nprogress

[github](https://github.com/rstacruz/nprogress)
```shell
npm i nprogress -s
npm i @types/nprogress -d

```

```html
<template>
    <div>
        <pre v-highlightjs='sourcecode'><code class='javascript' /></pre>
    </div>
</template>

<script setup lang="ts">

import { ref ,onMounted } from 'vue';
import NProgress from 'nprogress';
import 'nprogress/nprogress.css';

let sourcecode = ref('var hello = 1');

NProgress.configure({
    showSpinner:false
});


onMounted(() => {
    console.log('start');
    NProgress.start();
    console.log('start');
    
    setTimeout(() => {
        
        sourcecode.value = 'var fuck = 2';
        NProgress.done();
        console.log('done',sourcecode.value);
        
    }, 3000);
}
);
</script>
```


可定制进度条的具体形式，下面是默认的
```js
NProgress.configure({
	template: '<div class="bar" role="bar"><div class="peg"></div></div><div class="spinner" role="spinner"><div class="spinner-icon"></div></div>'
});
```
###  types/sortablejs

实现拖拽效果

```shell
npm i @types/sortablejs -s 
```

###  vuedraggable

基于sortable的vue的拖拽，[github](https://github.com/SortableJS/vue.draggable.next)

```shell
npm i -S vuedraggable@next 
```

### vue-splitpane

分割窗口

```shell
npm i splitpanes@next -s
```

### xlsx

```shell
npm i xlsx -s
```


### animejs

动画效果 [github](https://github.com/juliangarnier/anime/)


```shell
npm install animejs --save

```


###  normalize.css

在默认的HTML元素样式上提供了跨浏览器的高度一致性

```shell
npm install normalize.css -s
```

###  revogrid

一个简单的类似excel的类库

[📒 github](https://revolist.github.io/revogrid/)


```shell
npm i @revolist/revogrid --save;
npm i @revolist/vue3-datagrid
```