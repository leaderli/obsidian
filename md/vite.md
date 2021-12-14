---
icon: summary
---

## 快速入门

[📒 官网](https://vitejs.dev/)
```shell
npm init vite@latest

# 根据提示依次输入项目名，选择vue vue-ts
```

`vite.config.ts`
```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import path from "path";

// https://vitejs.dev/config/
export default defineConfig({
	
	//预先加载一些依赖
	optimizeDeps:{  
	    entries:path.resolve(__dirname,'src')  
	},
	plugins: [vue()],
	//服务器配置
	server: {
		//访问地址配置
		host: "0.0.0.0",
		//端口
		port: 3000,
		//是否使用https
		https: false,
		//启动后自动打开浏览器
		open: true,

		//代理配置，
		proxy: {
			"/api": {
				target: "",
				changeOrigin: true,
				rewrite: (path) => path.replace(/^\/api/, ""),
			},
		},

		hmr: {
			//是否屏蔽服务器保持
			overlay: false,
		},
	},
	//设置项目文件路径，以项目根目录文起点
	resolve: {
		//设置项目文件路径的别名
		alias: {
			"@": path.resolve(__dirname, "./src"),
		},
	},
});
```

然后启动
`npm run dev`

##  plugin-vue

[plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#example-for-passing-options-to-vuecompiler-dom)




## 添加mock

[[mockjs]]

[vite-plugin-mock](https://github.com/anncwb/vite-plugin-mock)

```shell
npm i  mockjs -D

npm i vite-plugin-mock -D

```


`vite.config.ts`

```js
import { viteMockServe } from 'vite-plugin-mock';


export default defineConfig(({ command }) => {
    return {
        plugins: [
            vue(),
            viteMockServe({
                // default
                mockPath: 'mock',
                logger: false,
                localEnabled: command === 'serve'
            }),

        ],
		server: {
			//代理配置，
			proxy: {
				"/api": {
					target: "",
					changeOrigin: true,
					rewrite: (path) => path.replace(/^\/api/, ""),
				},
				"/test": {
					target: "http://centos7:10001",
					rewrite: (path) => path.replace(/^\/test/, ""),
				},
			},
		}
    };
});

```


`mock/test.ts`

```js
// test.ts

import { MockMethod } from "vite-plugin-mock";
export default [
	{
		url: "/api/get",
		method: "get",
		response: ({ query }) => {
			return {
				code: 0,
				data: {
					name: "vben",
				},
			};
		},
	},
	{
		url: "/api/post",
		method: "post",
		timeout: 2000,
		response: {
			code: 0,
			data: {
				name: "vben",
			},
		},
	},
	{
		url: "/api/text",
		method: "post",
		rawResponse: async (req, res) => {
			let reqbody = "";
			await new Promise((resolve) => {
				req.on("data", (chunk) => {
					reqbody += chunk;
				});
				req.on("end", () => resolve(undefined));
			});
			res.setHeader("Content-Type", "text/plain");
			res.statusCode = 200;
			res.end(`hello, ${reqbody}`);
		},
	},
] as MockMethod[];

```



通过测试，发现`/api`被mock拦截了，而`/test`没有被拦截

```js
import axios from 'axios';
axios({
	url: "/api/post",

	method: "post",
	data: "{}",
	headers: { "Content-type": "application/json" },
}).then((res: {}) => {
	console.log(res);
});

axios({
	url: "/test",
	method: "get",
}).then((res: {}) => {
	console.log(res);
});
```

## 一些常见问题


````ad-info
title:Invalid Host header


`vue.config.js`
```javascriot
module.exports = {
  devServer: {
    disableHostCheck: true
  }
}
```
````




````ad-info
title:  compilerOptions

> runtime-core.esm-bundler.js:6589 [Vue warn]: Failed to resolve component: 
menubar-item If this is a native custom element, make sure to exclude it from component resolution via compilerOptions.isCustomElement.


因为组件`menubar-item`不存在，它可能是一个自定义的动态组件，将其加入编译选项中

```js
import vue from '@vitejs/plugin-vue';


export default defineConfig(({ command }) => {
    return {
        plugins: [
            vue({
                template: {
                    compilerOptions: {
						//需要精准定义为自己定义的标签
                         isCustomElement : (tag) => tag === 'menubar-item' 

                    }
                }
            }),
        ],
		
}		
```
````

^c9todk

````ad-info
> Error: spawn xdg-open ENOENT

关闭启动后用浏览器打开

```js
server:{
	open:false
}
```


````


## glob
[glob | Vite 官方中文文档](https://cn.vitejs.dev/guide/features.html#glob-import)

Vite 支持使用特殊的 `import.meta.glob` 函数从文件系统导入多个模块：

```js
const modules = import.meta.glob('./dir/*.js')
```

以上会被转译为
```js
// vite 生成的代码
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

你可以遍历 `modules` 对象的 key 值来访问相应的模块，匹配到的文件默认是懒加载的，通过动态导入实现

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```


如果想要直接引入，可以使用`import.meta.globEager` 代替：

```js
// ./modules/demo.ts
const demo = {
	a:1
}

export default demo
```
```js
const modules = import.meta.globEager('./modules/*.ts')
or (const path in modules) {
  	console.log(modules[path].default)//  {a:1}
  })
}
```

## svg

[vite-plugin svg](https://github.com/anncwb/vite-plugin-svg-icons)

根据`color`的颜色自动调整图标颜色，可以SVG文件中的属性更改

```html
<svg stroke="currentColor"> <svg>
```



图标大小调整 
```html
<svg viewBox="0 0 24 24"> <svg>
```

