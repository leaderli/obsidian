
## 快速入门

```shell
npm init vite@latest
npm run dev
```

## setup


语法糖，会被编译为`setup()`方法

```html
<script setup lang="ts">
</script>
```

1. [[vue#使用 props 传递数据|props]]
2. [[vue#emit|emit]]
3. [[vue#slot|slot]]
4. expose
5. 局部变量直接和原有的`data()`类似


以一个父子组件示例说明


`app.vue`

```html
<script setup lang="ts">
import { ref,onMounted } from 'vue';
import HelloWorld from './components/HelloWorld.vue'

const msg = ref('我是父组件')

const handleChange = (params: String)=>{
  console.log(params);
}

//需要与对应子组件的ref属性值相同
const  liRef = ref()

//挂载后执行
onMounted(()=>{
  //调用子组件的属性
  console.log(liRef.value.child)
  //调用子组件的方法
  liRef.value.child_method()
})

</script>

<template>
  <HelloWorld ref="liRef" :msg="msg" @on-change="handleChange" />
</template>

<style>
</style>
```


`HelloWorld.vue`
```html
<script setup lang="ts">


// 定义使用父组件的变量
const props = defineProps({
  msg:{
    type:String,
    default:()=>'默认值'
  }
})


//定义一个可以抛出的事件
const emit  = defineEmits(['on-change'])

const handleClick = ()=> {
  //抛出事件，附带参数
  emit('on-change','父组件方法被调用了')
}


//定义暴露给父组件的属性和方法
defineExpose({
  child:'我是暴露的子组件',
  child_method:()=>{
    console.log('我是暴露的子组件方法')
  }
})
	
	
const counter = ref(0)
//立即监听变量
watch(
  counter,
  (newValue, oldValue) => {
    console.log('The new counter value is: ' + counter.value)
  },
  { deep: true, immediate: true }
)
</script>

<template>
	<p>{{props.msg}}</p>
	<button @click="handleClick">点击调用父组件方法</button>
	<p>{{ counter }}</p>
	<button @click="counter++">增加</button>
</template>

<style>
</style>

```

## setup钩子

| 选项式API       | setup中使用       | 说明 |
| --------------- | ----------------- | ---- |
| beforeCreate    | Not needed*       |      |
| created         | Not needed*       |      |
| beforeMount     | onBeforeMount     |      |
| mounted         | onMounted         |      |
| beforeUpdate    | onBeforeUpdate    |      |
| updated         | onUpdated         |      |
| beforeUnmount   | onBeforeUnmount   |      |
| unmounted       | onUnmounted       |      |
| errorCaptured   | onErrorCaptured   |      |
| renderTracked   | onRenderTracked   |      |
| renderTriggered | onRenderTriggered |      |
| activated       | onActivated       |      |
| deactivated     | onDeactivated     |      |
|                |                   |      |


例如

```html
<script setup lang="ts">

import { onMounted, onUnmounted } from 'vue';

onMounted(() => {
  console.log('onMounted');
})

onUnmounted(() => {

  console.log('onUnmounted');
})
	
function created
</script>

<template>
  <p>test</p>
</template>

<style>
</style>


```
## 响应式数据

### ref
ref 让基础数据类型具备响应式

```js
import { ref } from 'vue'
const counter = ref(0)
```

`ref` 接收参数并将其包裹在一个带有 `value` property 的对象中返回，然后可以使用该 property 访问或更改响应式变量的值：

```js
import { ref } from 'vue'

const counter = ref(0)

watch(
  counter,
  (newValue, oldValue) => {
    console.log('The new counter value is: ' + counter.value)
  },
  { deep: true, immediate: true }
)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```
### reactive
reactive 让引用类型的数据具备响应式

```html
<template>
    <div>  {{ me.want }} </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';
let me = reactive({
    single:true,
    want:'暖的像火炉的暖男'
});
setTimeout(() => {
    me.want = '夏天容易化了';
},3000);
</script>
```

### toRefs、toRef
setup + ref + reactive 实现了数据响应式，不能使用 ES6 解构，会消除响应特性。所以需要 toRefs 解构，使用时，需要先引入。

```html
<template>
    <div>  {{ want }} </div>
</template>

<script setup lang="ts">
	
import { reactive ,toRefs } from 'vue';
let me = reactive({
    single:true,
    want:'暖的像火炉的暖男'
});
setTimeout(() => {
    me.want = '夏天容易化了';
},3000);

const { want } = toRefs(me);

</script>
```


toRef尝试从变量中解构，如果没有则创建一个

```js
const love = toRef(me,'love')
```
## withDefaults


一种写法方便的写法，替代defineProps默认写法
```js

// 定义使用父组件的变量
// const props = defineProps({
//   msg: {
//     type: String,
//     default: () => '默认值'
//   }
// })


withDefaults(defineProps<{
  msg?: string,
  title?: string
}>(), {
  msg: 'hello',
  title: 'title'
})
```

defineProps设置复杂的泛型

```js
import { PropType } from 'vue'

defineProps({
  menuList: {
    type: Object as PropType<Array < IMenubarList >>,
    default: []
  }
})
```

## 自定义指令

[[vue#自定义指令]]
```js
app.directive('focus',{
 mounted(el){
  el.focus()
 }
})
```

```html
<input type="text" v-focus />
```

### 局部自定义指令

略

## 使用vuex

可参数vue2中的[[vue#vuex|vuex]]


`store/index.ts`
```js
import { InjectionKey } from "vue";
import { useStore as baseUseStore, createStore, Store } from "vuex";

//声明state的类型
export interface State {
	username: string;
}

//定义注入类型,类似.d.ts文件，是用于typescript推断类型使用
export const key: InjectionKey<Store<State>> = Symbol();

//定义store
export const store = createStore<State>({
	state: {
		username: "li",
	},
	mutations: {},
	getters: {},
	actions: {},
});

export function useStore() {
	return baseUseStore(key);
}
```

`main.ts`引入vuex

```js
import { createApp } from 'vue'
import App from './App.vue'
import { key, store } from './store'

const app = createApp(App)
//引入vuex
app.use(store,key)
app.mount('#app')

```

使用

```html
<script setup lang="ts">
	
import { useStore } from '../store';
const store = useStore()
console.log(store.state.username)
	
</script>

<template>
	  <p>{{ store.state.username }}</p>
</template>

<style>
</style>
```


对于使用module的vuex可以做如下定义


`store/modules/user.ts`

```js
import { createStore } from "vuex";

//声明state的类型
interface User {
	username: string;
}

//定义store
export const user = createStore<User>({
	state: {
		username: "li",
	},
	mutations: {},
	getters: {},
	actions: {},
});

```

`store/index.ts`
```js
import { InjectionKey } from "vue";
import { user } from "./modules/user";

import { useStore as baseUseStore, createStore, Store } from "vuex";

//声明state的类型
export interface State {
	// username: string;
	user: typeof user.state;
}

//定义注入类型
export const key: InjectionKey<Store<State>> = Symbol();

//定义store
export const store = createStore<State>({
	mutations: {},
	getters: {},
	actions: {},
	modules: {
		user: user,
	},
});

export function useStore() {
	return baseUseStore(key);
}

```

在component中使用
```html
<script setup lang="ts">
	
import { useStore } from '@/store';
const store = useStore()
console.log('hello user ', store.state.user)
	
</script>
<template>
  <p>{{ store.state.user.username }}</p>
</template>
```
## 使用router

参考[[vue#vue-route|router]]

```shell
npm install vue-router@4
```

准备相关页面

`pages/about/index.vue`
```html
<template>
    <h2>about页面</h2>
</template>
```

`pages/home/index.vue`
```html
<template>
    <h2>home页面</h2>
</template>
```

`components/layout/header/index.vue`
```html
<template>
    <div class="action">
        <h2 @click="handleClick(1)">首页</h2>
        <h2 @click="handleClick(0)">关于</h2>
    </div>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";

const router = useRouter();

//切换路由
const handleClick = (num: number) => {

    if (num) {
        router.push({
            name: "home",
        });
    } else {
        router.push({
            name: "about",
        });
    }
};
</script>

<style>
.action {
    display: flex;
}

h2 {
    padding: 0px 10px;
    cursor: pointer;
}
h2:hover {
    color: red;
}
</style>

```

`components/layout/index.vue`

```html
<template>
    <Header/>
    <router-view></router-view>
</template>
<script setup lang="ts">
import Header from './header/index.vue'
</script>
```

`main.ts`

```js
import { createApp } from "vue";
import App from "./App.vue";

import router from "./router";

const app = createApp(App);
//引入router
app.use(router);
app.mount("#app");

```


编辑路由规则页面

`router/index.ts`
```js
import { createRouter, createMemoryHistory } from "vue-router";

import LayOut from "../components/layout/index.vue";
const routes = [
	{
		path: "/",
		component: LayOut,
		redirect: "",
		children: [
			{
				path: "/home",
				name: "home",
				component: () => import("../pages/home/index.vue"),
				meta: {
					title: "首页",
					icon: "",
				},
			},
			{
				path: "/about",
				name: "about",
				component: () => import("../pages/about/index.vue"),
				meta: {
					title: "关于",
					icon: "",
				},
			},
		],
	},
];

const router = createRouter({
	history: createMemoryHistory(),
	routes,
});

export default router;

```

## 集成element-plus

[官方文档](https://element-plus.gitee.io/zh-CN/guide/quickstart.html)

```shell
npm  install element-plus --save
# 图标库
npm install @element-plus/icons
```


`main.ts`

```js
import { createApp } from "vue";
import App from "./App.vue";
import ElementPlus from "element-plus";
import "element-plus/dist/index.css";

const app = createApp(App);
app.use(ElementPlus);
app.mount("#app");
```

```html
<template>
    <h2>home页面</h2>
    <el-button>I am ElButton</el-button>
    <div>
        <el-icon :size="size" :color="color">
            <edit></edit>
        </el-icon>
    </div>
</template>

<script setup lang="ts">
import { ElButton } from 'element-plus'
//局部使用图标,可以全局注册到vue中	
import { Edit } from '@element-plus/icons'

const size = 20
const color = 'green'
</script>
```

##  安装sass

```shell
npm install node-sass sass-loader sass
```

`assests/styles/global.scss`

```sass
$color-primary:#007aff;
```

```html
<template>
    <h2 class="test-sass-color">home页面</h2>
</template>

<script setup lang="ts">
</script>

<style lang="scss" scoped>
.test-sass-color {
    color: $color-primary;
}
</style>
```

`vite.config.ts`新增配置

```js
export default defineConfig({
	// ...
	css: {
		preprocessorOptions: {
			scss: {
				additionalData: '@import "@/assets/styles/global.scss";',
			},
		},
	},
}
```


## 泛型

```js
import { DefineComponent } from "vue";
// vue组件的泛型
type component = DefineComponent<{}, {}, any>
```


```js
import { DefineComponent } from "vue";

type  define_component= DefineComponent<{}, {}, any>
type typeOf_import_vue = Promise<typeof import('*.vue')>
type func_typeOf_import_vue = ()=>typeOf_import_vue
type component = define_component|typeOf_import_vue|func_typeOf_import_vue

export interface IMenubarList {
    parentId?: number | string
    id?: number | string
    name: string
    path: string
    redirect?: string | {name: string}
    meta: {
        icon: string
        title: string
        permission?: string[]
        activeMenu?: string // 路由设置了该属性，则会高亮相对应的侧边栏
        noCache?: boolean // 页面是否不缓存
        hidden?: boolean // 是否隐藏路由
        alwaysShow?: boolean // 当子路由只有一个的时候是否显示当前路由
    }
    component:component
    children?: Array<IMenubarList>
}
```


```js
import LayOut from "@/components/layout/index.vue";
import home from "@/views/home/index.vue"


export const allowRouter: Array<IMenubarList> = [
	{
		name: "",
		path: "/",
		meta: {
			title: "404",
			icon: "",
		},
		component: LayOut,
		redirect: '',
		children: [
			{
				path: "/home",
				name: "home",
				component: home,
				meta: {
					title: "首页",
					icon: "",
				},
			},
			{
				path: "/about",
				name: "about",
				component: import("@/views/about/index.vue"),
				meta: {
					title: "关于",
					icon: "",
				},
			},
			{
				path: "/404",
				name: "404",
				component: () => import("@/views/404/404.vue"),
				meta: {
					title: "404",
					icon: "",
				},
			},
		],
	},
	{
		path: "/login",
		name: "login",
		component: () => import("@/views/login/index.vue"),
		meta: {
			title: "login",
			icon: "",
		},
	},
];
```





## vue-tsc
检测类型的包

##  一些常见问题
> node_modules/@antv/x6/lib/view/tool.d.ts:76:30 - error TS2503: Cannot find namespace 'JQuery'.

忽略 [[#vue-tsc]] 检测依赖包即可

```json
{
  "script": {
    "build": "vue-tsc --noEmit --skipLibCheck && vite build"
  }
}

```