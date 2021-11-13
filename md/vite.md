## 快速入门

```shell
npm init vite@latest

# 根据提示依次输入项目名，选择vue vue-ts

```

`vite.config.ts`
```shell
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  server:{
    host:'0.0.0.0'
  }
})

```

然后启动
`npm run dev`

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