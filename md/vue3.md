
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

//需要与ref属性值相同
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


获取`$el`属性，得到当前组件的dom

```js
liRef.value.$el.getBoundingClientRect()
// 获取子组件的ref
liRef.value.child_ref
```

`HelloWorld.vue`
```html
<script setup lang="ts">

import { watch } from 'vue'


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
	<p ref='child_ref'>{{props.msg}}</p>
	<button @click="handleClick">点击调用父组件方法</button>
	<p>{{ counter }}</p>
	<button @click="counter++">增加</button>
</template>

<style>
</style>

```

### ts特有的语法糖
```js
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 有默认值的props

interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

### 子组件修改父组件属性的方法

使用`v-model`修饰符，类似与2的`sync`修饰符，具体 [📒 参考](https://v3.vuejs.org/guide/migration/v-model.html#migration-strategy)
```html
<template>  
 <h5>{{ msg }}</h5>  
 <hello v-model:msg='msg' />  
</template>  
  
<script setup lang='ts'>  
import hello from './hello.vue';  
import { ref } from 'vue';  
const msg = ref('text');  
</script>
```

`home.vue`

```html
<template>  
 <h2>{{ msg }}</h2>  
</template>  
  
<script setup lang='ts'>  
import { onMounted } from 'vue';  
  
const props = defineProps({  
  
 msg:{  
 type:String  
 }}  
);  
  
const emit = defineEmits(['update:msg']);  
  
  
onMounted(() => {  
    emit('update:msg','123123');  
});  
</script>  
```

## setup钩子

![[Pasted image 20211209094645.png|600]]

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
import { ref ,watch } from 'vue'

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
    default: function() {return []}
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

## 使用Pinia

[官方文档](https://pinia.esm.dev/core-concepts/)

```shell
npm install pinia -s
```


`store/index.ts`
```js
import { createPinia } from 'pinia';  
export const pinia = createPinia();
```

`store/modules/User.ts`

```js
import { defineStore } from 'pinia';  
  
// 泛型接口
import { User } from '@/type/user';  

  
export const defineUser = defineStore({  
    id: 'user',  
    state: (): User => ({  
        username: 'li',  
        password: 'li'  
 }),  
    getters: {
		full(state){
			return state.username+":"+state.password
		}
	},  
    actions: {
		changePassword(password){
			this.password = password
		}
	}  
});
```

注册
`main.ts`
```js
import { createApp,Directive } from 'vue';
import { pinia} from './store'

const app = createApp(App);
app.use(pinia)
```

使用
```html
<template>  
    <h2>about页面</h2>  
    <p>{{ user.username }}</p>  
    <p>{{ user.full}}</p>  
</template>  
  
<script setup lang="ts">  
import { defineUser } from '@/store/modules/user';  
import { User } from '@/type/user';
  
const user:User = defineUser();  
	
user.changePassword('345'):
  
</script>
```
## inject provide
用于父组件与子组件或孙组件注入数据，相对于`props`和`emit`，它支持多层嵌套

父组件
```html
<script setup lang="ts">
import { provide } from "vue"
provide('info',"值")
</script>
<template>
	<child/>
</template>
```

子组件
```html
<script setup lang="ts">
import { inject} from "vue"
	
const info = inject('info')
</script>
<template>
  <p>{{ info}}</p>
</template>
```


在父组件中定义[[#响应式数据]]，也可以定义readonly，限制子组件修改数据


```html
<script setup lang="ts">
import { provide ,readonly} from "vue"

//发布
let info = ref("今天你学习了吗？")
const changeInfo = (val)=>{
 info.value = val
}
provide('info',readonly(info))
//提供一个接口用于改变数据
provide('changeInfo',changeInfo) //订阅
</script>
<template>
	<child/>
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

[Sass基础教程 Sass快速入门 Sass中文手册 | Sass中文网](https://www.sass.hk/guide/)

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



响应式泛型类型

```js
import {ref,Ref} from 'vue'

let a:Ref<number> = ref<number>(1)

let b:UnwrapNestedRefs<object> = reactvie<object>({})
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

## 动态组件

[官方文档](https://v3.vuejs.org/guide/component-dynamic-async.html#dynamic-async-components)

```html
 <component :is='menu.meta.icon' class='icons'/>
```

也可以使用`resolveComponent`通过 [[#render]]的方式动态渲染



## render

> 目前还没找到支持 vue3 composition api 的写法

```html
<script lang='ts'>
	
import {defineComponent,h,resolveComponent} from 'vue'
	
export default defineComponent({
	render(){
		return  h(resolveComponent('el-button'),{},()=>[])
	}
})
	
</script>
```

## eslint

[eslint-vue](https://eslint.vuejs.org/user-guide/#usage)

```shell
npm install --save-dev eslint eslint-plugin-vue @typescript-eslint/parser @typescript-eslint/eslint-plugin 
```

`package.json`

```json
"scripts": {
	"lint": "eslint . --ext .ts,vue"
},
```

`.eslintrc.js`
```js
module.exports = {
    parser: 'vue-eslint-parser',
    parserOptions: {
        parser: '@typescript-eslint/parser',
        sourceType: 'module',
        ecmaFeatures: {
            jsx: true,
            tsx: true
        }
    },
    env: {
        browser: true,
        node: true,
        'vue/setup-compiler-macros': true // 支持setup语法糖

    },
    plugins: ['@typescript-eslint'],
    extends: [
        'plugin:@typescript-eslint/recommended',
        'plugin:vue/vue3-recommended'
    ],
    rules: {
        'vue/singleline-html-element-content-newline': 'off',
        'vue/multiline-html-element-content-newline': 'off',
        'vue/html-indent': ['error', 4],
        indent: ['error', 4], // 4行缩进
        'vue/script-indent': ['error', 4],
        quotes: ['error', 'single'], // 单引号
        'vue/html-quotes': ['error', 'single'],
        semi: ['error', 'always'], // 使用分号
        'space-infix-ops': ['error', { int32Hint: false }], // 要求操作符周围有空格
        'no-multi-spaces': 'error', // 禁止多个空格
        'no-whitespace-before-property': 'error', // 禁止在属性前使用空格
        'space-before-blocks': 'error', // 在块之前强制保持一致的间距
        'space-before-function-paren': ['error', 'never'], // 在“ function”定义打开括号之前强制不加空格
        'space-in-parens': ['error', 'never'], // 强制括号左右的不加空格
        'space-infix-ops': 'error', // 运算符之间留有间距
        'spaced-comment': ['error', 'always'], // 注释间隔
        'template-tag-spacing': ['error', 'always'], // 在模板标签及其文字之间需要空格
        'no-var': 'error',
        'prefer-destructuring': [
            'error',
            {
                // 优先使用数组和对象解构
                array: true,
                object: true
            },
            {
                enforceForRenamedProperties: false
            }
        ],
        'comma-dangle': ['error', 'never'], // 最后一个属性不允许有逗号
        'arrow-spacing': 'error', // 箭头函数空格
        'prefer-template': 'error',
        'template-curly-spacing': 'error',
        'quote-props': ['error', 'as-needed'], // 对象字面量属性名称使用引号
        'object-curly-spacing': ['error', 'always'], // 强制在花括号中使用一致的空格
        'no-unneeded-ternary': 'error', // 禁止可以表达为更简单结构的三元操作符
        'no-restricted-syntax': [
            'error',
            'WithStatement',
            'BinaryExpression[operator="in"]'
        ], // 禁止with/in语句
        'no-lonely-if': 'error', // 禁止 if 语句作为唯一语句出现在 else 语句块中
        'newline-per-chained-call': ['error', { ignoreChainWithDepth: 2 }], // 要求方法链中每个调用都有一个换行符
        // 路径别名设置
        'no-submodule-imports': ['off', '/@'],
        'no-implicit-dependencies': ['off', ['/@']],
        '@typescript-eslint/no-explicit-any': 'off', // 类型可以使用any
        '@typescript-eslint/no-non-null-assertion': 'off',
        'vue/multi-word-component-names': 'off',
        'prettier/prettier': 'off',
        'no-undef':'error', // 未定义的变量
        'vue/script-setup-uses-vars': 'error' // 

    }
};

```

`.eslintignore`

```txt
build/*.js
src/assets
public
dist
```


部分代码不使用eslint检测

```js
type define_component = DefineComponent<unknown, unknown, any>; // eslint-disable-line


/* eslint-disable no-console, no-control-regex*/
console.log('JavaScript debug log');
console.log('eslint is disabled now');
```

## stylelint

```shell
npm i stylelint stylelint-config-standard -D

```


```js
module.exports = {
    processors: [],
    plugins: [],
    extends: "stylelint-config-standard", // 这是官方推荐的方式
    ignoreFiles: ["node_modules/**", "dist/**"],
    rules: {
        "at-rule-no-unknown": [ true, {
            "ignoreAtRules": [
                "responsive",
                "tailwind"
            ]
        }],
        "indentation": 4,        // 4个空格
        "selector-pseudo-element-no-unknown": [true, {
            "ignorePseudoElements": ["v-deep"]
        }],
        "value-keyword-case": null
    }
}
```



## 示例

```js
import {computed} from "vue";

const name = computed(()=>{
	return 123;
})
```

##  组件网站

[vuescript](https://www.vuescript.com/)