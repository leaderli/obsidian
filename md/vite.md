[官方配置](https://vitejs.dev/config/)
## 快速入门

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


## 添加mock

[vite-plugin-mock](https://github.com/anncwb/vite-plugin-mock)

```shell
npm i  mockjs -S

npm i vite-plugin-mock -D

```


`vite.config.ts`

```js
export default defineConfig(({ command, mode }) => {
	return {
		plugins: [
			vue(),
			viteMockServe({
				// mock的ts文件的路径
				mockPath: "mock",
				localEnabled: command === "serve",
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
	}
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


Invalid Host header


`vue.config.js`
```javascriot
module.exports = {
  devServer: {
    disableHostCheck: true
  }
}
```