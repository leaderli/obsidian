[官方文档](https://panjiachen.github.io/vue-element-admin-site/zh/guide/)
## 安装

```shell
$ git clone git@github.com:PanJiaChen/vue-element-admin.git

$ npm install
```



## 模拟报文
[mock](https://panjiachen.github.io/vue-element-admin-site/zh/guide/essentials/mock-api.html)


默认情况下本地的请求会被代理到`http://localhost:${port}/mock`下，如果想要调整为自己的mock地址可以修改proxy

`vue.config.js`
```javascript
proxy: {
  // change xxx-api/login => mock/login
  // detail: https://cli.vuejs.org/config/#devserver-proxy
  [process.env.VUE_APP_BASE_API]: {
    target: `http://localhost:${port}/mock`,
    changeOrigin: true,
    pathRewrite: {
      ['^' + process.env.VUE_APP_BASE_API]: ''
    }
  }
},
after: require('./mock/mock-server.js')
```


```ad-warning
当我们设置了proxy代理，但axios请求一直pendding的时候，一般有可能是mock将请求拦截了。我们只要在vue.config.js将before的mock删除，或更改为after
```