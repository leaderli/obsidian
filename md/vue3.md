
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


## ref

我们可以通过一个新的 `ref` 函数使任何响应式变量在任何地方起作用

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