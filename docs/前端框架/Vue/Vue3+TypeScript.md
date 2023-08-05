## 基础知识

### 导出与导入

```js
// export 和 export default导出的区别
export可以在一个文件中导出多个，而export default在一个文件中只能导出一个
export导出的是具有名字的，引入的时候可以解析引入，而export default 导出是匿名导出

// 使用
// 导出 index.ts
export function ExportFun (): string { return '这是export导出' }
export default function (): string { return '这是export default导出' }
// 导入
// export default
import ExportDefault from "@/utils"
// export
import { ExportFun } from "@/utils"


// 在一个文件夹中声明了多个vue文件，如何统一导出
|- components
|--- client
|----- One.vue  (不同的vue组件)
|----- Two.vue
|----- Three.vue
|----- index.ts  (统一出口)
// index.ts 配置
规范： export { default as 自定义组件名称 } from '该组件路径'
export { default as OneVue } from './One.vue'
export { default as TwoVue } from './Two.vue'
export { default as ThreeVue } from './Three.vue'


```

### setup 组合式 API

```js
// Vue3中的组合式API

// ref
// ref()函数用来根据给定的值创建一个响应式的数据对象，ref()函数调用的返回值是一个对象，这个对象上只包含一个value属性
// 导入
import { ref } from "vue
// 定义
setup(){
  let c = ref(10); //初始化值为10
  return {c}
}
// 使用
<template>
  <div>
  	{{c}}
  </div>
</template>

// reactive
// reactive()函数接收一个普通对象，返回一个响应式的数据对象
// 导入
import { reactive } from "vue"
// 定义
setup(){
  const state = reactive({
    name:"WEB架构师"
  })
  return state
}
// 使用
<template>
  <div>
  	{{name}}
  </div>
</template>

// ref与reative的不同
// reactive的用法与ref的用法相似，也是将数据变成响应式数据，当数据发生变化时UI也会自动更新。不同的是ref用于基本数据类型，而reactive是用于复杂数据类型

// computed
// computed()用来创建计算属性，computed()函数的返回值是一个 ref 的实例，计算属性可以传入变量 通过return函数的形式使用
// 导入
import {computed} from "vue";
// 调用
setup(){
  let num = 20
  // 转化
  const moneyConversion = computed(()=>num=>{
    return `￥${num.toFixed(2)}`
  }}

  return {num,moneyConversion}
}
// 使用
<template>
	// 利用计算属性 转化为 ￥20.00格式
  {{moneyConversion(num)}}
</template>

// watch
// watch() 函数用来监视某些数据项的变化，从而触发某些特定的操作,可以设置监听或者关闭监听
// 导入
import {watch,reactive,toRefs} from "vue"
// 定义
setup(){
  const data = reactive({
  	num:0,
  	click(){
  		this.num++
      // 当num>=10时，关闭监听
  		if(this.num>=10){
  			stop()
  		}
  	},
  })
  let stop = watch(()=>state.num,()=>{
  	console.log(“num发生了变化”)
  })
  return {...toRefs(data)}
}
// 使用
<template>
	<h2>{{num}}</h2>
	<button @click="click"></button>
</template>

// 声明周期钩子
// dom加载后 数据更新后 组件卸载后
import { onMounted, onUpdated, onUnmounted } from "vue";
setup(){
  onMounted(()=>{
    console.log("onMounted")
  })
  onUpdated(()=>{
    console.log("onUpdated")
  })
  onUnmounted(()=>{
    console.log(onUnmounted)
  })
}

// 新旧对比
beforeCreate -> use setup()
created -> use setup()
beforeMount -> onBeforeMount
mounted -> onMounted
beforeUpdate -> onBeforeUpdate
updated -> onUpdated
beforeDestroy -> onBeforeUnmount
destroyed -> onUnmounted
errorCaptured -> onErrorCaptured


// Suspense 异步组件
// Suspense组件用于在等待某个异步组件解析时显示后备内容。1、在页面加载之前显示加载动画 2、显示占位符内容 3、处理延迟加载的图像
// 使用
<Suspense>
  <template #default>
    <List />
  </template>
	<template #fallback>
  	<div>loading......</div>
	</template>
</Suspense>

// router和route
// 由于setup中没有this，因此可以通过 useRouter() 和 useRoute()获取
import { useRouter,useRoute } from "vue-router"
setup(){
  const router = useRouter()
  const route = useRoute()
  // 路由跳转
  router.push("./home")
  // 获取当前路由传入id
  route.params.id
}


// props
// 用户父子组件传值
// 父组件
<childrenVue :name="name" :age="age"></childrenVue>
setup(){
  const userInfo = reactive({
    name:"WEB架构师",
    age:22
  })
  return {...toRefs(userInfo)}
}

// 子组件
<template>
	 <h2>name:{{name}},age:{{age}}</h2>
</template>
<script>
export default{
props:{
  name:String,
  age:Number
}
setup(props){
  const state = reactive({
    name:props.name,
    age:props.age
  })
  return {...toRefs(state)}
}
}

</script>

// vuex使用
//导入vuex
import {useStore} from "vuex"
setup(){
  // 定义store
  const store = useStore()
  // 使用state数据 获取响应式数据
  const num = computed(() => store.state.isFalse)
  // 使用mutations方法
  const tap = store.commit("tapIsFalse", 123) //前面为方法名 后面为传入的参数
  return {num,tap}

  // 也可以合在一起
  const vuex = reactive({
    num:computed(()=>store.state.isFalse),
      tap(){
        store.commit("tapIsFalse",123)
      }
  })
  return {...toRefs(vuex)}
}
// 使用actions方法
store.dispatch("getListData",_id)
// 在actions方法中调用mutations方法
actions: {
  async accountLogin({ commit }, loginData: AdminLoginForm) {
    commit('mutations方法名称', '传入的参数')
    commit('updateListData', loginData)
  }
}



<script>
// 导入二次封装的axios
import axios from "@/utils/axios.js"
// 导入需要的内容
import {computed, reactive, toRefs} from "vue"
export default{
	setup(){
    // 定义一个响应式数据，比如对象。列表
    const state = reactive({
      // 数据列表
      listData:[],
      // 用户信息
      userInfo:{},
      // 当前下标
      index:0
    })
  }
}
</script>
```

### script setup

```js
// Vue3.2 setup语法糖 <script setup></script>
在里面注册的组件不再需要导出，并且在里面的定义的任何变量 不再需要return导出，更接近于原生开发

// setup中组件之间传值

// 子组件
<template>
  这是Text.vue组件
  传入的数据为：{{props.mes}}
</template>
<script lang='ts' setup>
import { defineProps, defineEmits } from "vue"
// 定义props传入数据类型
interface Props{
  mes: string
  data: {}
}
// 获取传入的数据
const props = defineProps<Props>()
console.log(props.data)
</script>


// 父组件
<template>
  <h1>这是用户界面Home页面</h1>
	// 向子组件传递数据
  <TextVue :mes="'传入的数据'" :data="data"></TextVue>
</template>
<script lang='ts' setup>
import TextVue from '../Text.vue';
const data = {
  code: 0,
  mes: "文本数据",
  data: "这里存放的是数据"
}
</script>


// 使用emits子向父组件发送事件
// 父组件
<template>
  <h1>这是用户界面Home页面</h1>
	// 接受子组件发送的事件
  <TextVue @change='updateMes'></TextVue>
</template>

<script lang="ts" setup>
  const updateMes = (...rest) => {
    console.log('子组件发送了事件', rest)
  }
}
</script>

// 子组件
<template>
  <h1>这是子组件页面</h1>
	// 向父组件传递数据
	<button @click='handleClick'></button>
</template>

<script lang="ts" setup>
import { defineEmits } from 'vue'

// 注册emits事件
const emits = defineEmits(['change'])

const handleClick = () => {
  // 发射事件
	emits('change', '123')
}
</script>


// 并且在script setup里面可以直接使用await
<script lang="ts" setup>
  const sss = ():Promise<any> => {
  return new Promise((resolve, reject) => {
    resolve("123")
  })
}
const data = await sss() // "123"
</script>

```

### 组件的 v-model

```js
// v-model不仅可以使用在input等表单元素上，还可以在组件之间进行传值

// 父组件
<template>
  // 父组件使用v-model
	<TextVue v-model='message'></TextVue>
	// 等效于 默认双向绑定值为modelValue
	<TextVue :model-value="message" @update:model-value="message = $event"></TextVue>
	// 数据双向改变
	<input v-model='message' />
</template>
<script lang='ts' setup>
import { TextVue } from '@/components/dome'
import { ref } from 'vue'

const message = ref<string>('message')
</script>

// 子组件
<template>
  {{ props.modelValue }}
  // 数据双向改变
  <input v-model='mes' />
</template>
<script lang='ts' setup>
import { ref, defineProps, defineEmits, } from 'vue'
// 注意 传过来的是 model-value，而不是message
interface Props{
  modelValue: string
}
// 接受传递过来的modelValue
const props = defineProps<Props>()
// Props只能单向传递，但是可以使用emits将改变后的值发出去
const emits = defineEmits(['update:modelValue'])
// 利用计算属性 响应式绑定输入框内容，并且利用set将改变后的值用emits发出去
const mes: any = computed({
  set(newVal: string){
    emits('update:modelValue', newVal)
  },
  get(){
    return props.modelValue
  }
})
</script>


// 绑定多个v-model

// 父组件
<template>
  //  在v-model:后面加上一个名称 可以改变该props的值 并且会改变emits的值
	<TextVue v-model:message='message'></TextVue>
	// 数据双向改变
	<input v-model='message' />

  <TextVue v-model:title='title'></TextVue>
	<input v-model='title' />
</template>
<script lang='ts' setup>
import { TextVue } from '@/components/dome'
import { ref } from 'vue'

const message = ref<string>('message')
const title = ref<string>('title')
</script>

// 子组件
<template>
  // 数据双向改变
  <input v-model='mes' />
</template>
<script lang='ts' setup>
import { ref, defineProps, defineEmits, computed } from 'vue'

interface Props{
  message: string
  title: string
}
// 接受传递过来的modelValue
const props = defineProps<Props>()
// Props只能单向传递，但是可以使用emits将改变后的值发出去
const emits = defineEmits(['update:message', 'update:title'])
// 利用计算属性 响应式绑定输入框内容，并且利用set将改变后的值用emits发出去
const mes: any = computed({
  set(newVal: string){
    emits('update:message', newVal)
  },
  get(){
    return props.message
  }
})

const tit = computed({
  set(newVal){
    emits('update:title', newVal)
  },
  get(){
    return props.title
  }
}

</script>

```

### vue3 配置跨域

```js
// 开发环境下使用 Proxy 配置代理，
// 生产环境下使用  Nginx配置代理

// 开发环境 配置多个跨域

// axios.ts
const baseUrl = process.env.NODE_ENV === 'development' ? '/api' : ''

axios.defaults.baseURL = baseURL

// vue.config.js
module.exports = {
  devServer: {
    open: true,
    port: 5000,
    proxy: {
      '/api/ipJson': {
        target: 'http://whois.pconline.com.cn/ipJson.jsp?json=true',
        changeOrigin: true,
      },
      '/api': {
        target: 'http://huangjianxin.cn',
        // ws: true,
        changeOrigin: true,
        pathRewrite: {
          '^/api': '',
        },
      },
    },
  },
}

// 使用
axios.get('/api/home/matters/find').then(({ data }) => {
  console.log(data)
})
```

### 插槽 slot

```js
// 好比我们移动中常见的顶部，可能会有搜索框 可能会有返回框 但是有的页面有没有，这个时候我们可以使用插槽，也叫占位符 即预留的位置

// 插槽可以分为 匿名插槽 具名插槽  作用域插槽

// 匿名插槽 即没有规定名字的插槽 这样虽然可以插入，但是无法知道是在哪一个slot位置插入
// 父组件 App.vue
<template>
	<SlotVue>
  	<h3>这是插入的内容</h3>
  </SlotVue>
</template>
<script setup lang='ts'>
import { SlotVue } form './components/dome'
</script>

// 子组件 src/components/dome/Slot.vue
<template>
  <!-- 插槽 -->
	<slot></slot>
	<h2>Slot.vue组件</h2>
	<!-- 插槽 -->
	<slot></slot>
</template>
// 插入完插槽之后的页面
<template>
  <h3>这是插入的内容</h3>
	<h2>Slot.vue组件</h2>
	<h3>这是插入的内容</h3>
</template>


// 由上可知，匿名插槽的弊端在于当我们想插入下面的插槽的时候，由于我们是匿名插槽，因此我们无法准确插入下面的插槽，因此会在有插槽的地方全部插入

// 具名插槽 带名字的插槽
// 注意： 一个不带name的插槽会带有隐含的名字default
// 定义名字 <slot name='left'></slot>
// 父组件使用方法一 <template v-slot:left></template>
// 父组件使用方法二 <template #left></template>
// 父组件 App.vue
<template>
	<NavBarVue>
  	<!-- 给名字为left	的插槽插入内容 -->
  	<template #left>
    	<button>左侧的按钮</button>
    </template>
		<template #center>
    	<h2>中间的标题</h2>
    </template>
		<template #right>
    	<i>右侧的i元素</i>
    </template>
  </NavBarVue>
</template>
<script setup lang='ts'>
import { NavBarVue } form './components/dome'
</script>

// 子组件 src/components/dome/NavBar.vue
<template>
  <div class='left'>
  	<slot name='left'></slot>
  </div>
  <div class='center'>
    <slot name='center'></slot>
  </div>
  <div class='right'>
    <slot name='right'></slot>
  </div>
</template>



// 作用域插槽
// 渲染作用域：
// 在vue中，父模板所有内容都是在父模板编译的，子模板所有内容都是在子模版编译的，因此我们在父模板使用子模版时无法使用子组件的内容
// 作用域插槽：
// 当一个组件用来渲染一个数组元素时，我们使用插槽，并且希望插槽中没有显示没项目的内容

// 父组件 App.vue
<template>
	<NavBarVue :names='names'>
    // 接受子组件传递过来的数据 全部放在scope中
  	<template #default="scope">
      <h2>{{ scope.item }} - {{ scope.index }}</h2>
    </template>
  </NavBarVue>
</template>
<script setup lang='ts'>
import { ShowNamesVue } from '@/components/dome'

const names = ['web', 'js', 'axios', 'react']
</script>

// 子组件 src/components/dome/ShowNames.vue
<template>
  <div id="showName">
    <template v-for="(item, index) in props.name" :key="item">
      // 传递给父组件子组件的内容
      <slot :item="item" :index="index"></slot>
    </template>
  </div>
</template>
<script lang="ts" setup>
import { defineProps } from 'vue'

interface Props{
  name: Array<string>
}

const props = defineProps<Props>()
</script>



```

### 自定义全局指令

```js
// 在vue中自定义全局组件，比如v-focus页面加载自动获取搜索框焦点

// 在main.ts中进行注册和使用（不推荐使用）
const app = createApp(App)
// 自定义全局指定
// 第一个参数 自定义指令的名称， 使用：前面加上v- ,第二参数为对象，在各个生命周期进行的操作
app.directive('focus', {
  // el代表当前获取的元素
  mounted(el){
    console.log('v-fous执行')
    el.focus()
  }
})

app.mount('#app')


// 封装使用 （推荐使用）
// 在src下新建directive文件夹，然后在文件夹新建 index.ts和v-format-time.ts(或其它指令)

// 在src/directive/index.ts中
// 引入app类型
import type { App } from 'vue'

// 调用自定义指定 v-formTime
import registerFormatTime from './v-format-time'

export default function (app: App<Element>){
 	// 注册v-format-time组件
  registerFormatTime(app)
}


// 在src/directive/v-format-time.ts中
import type { App } from 'vue'
import dayjs, { relativeTime } from '@/utils/date-time'

export default function vFormatTime(app: App<Element>){
  // 默认返回格式
  let defaultStyle = 'YYYY-MM-DD HH:mm:ss'
  app.directive('format-time', {
    created(el, bindings){
      if(bindings.value) {
        defaultStyle = bindings.value
      }
    },
    // el是作用的元素 bindings是指令的参数和配置
    mounted(el, bindings){
      // 获取元素的文本内容
      console.log(el.textContent)
      // 获取指令的参数内容
      console.log(bindings.value)
      // 获取指令的配置
      console.log(bindings.modifiers)

      const textContent = el.textContent
      // string 转 number
      let textNumber = parseInt(textContent)
			// Date 转 number
      // let textNumber = new Date(textContent).getTime()

      // 如果时间错不是毫秒 转化为毫秒
      if(textContent.length <= 10){
        textNumber = textNumber * 1000
      }

      // 默认返回时间格式
      // @ts-ignore
      el.textContent = dayjs(textNumber).format(defaultStyle)

      // 获取是否传入指令配置
      Object.keys(bindings.modifiers).forEach(item => {
        // 如果为查询倒数几天
        if(item === 'relative'){
          // @ts-ignore
          el.textContent = relativeTime(textNumber)
        }
      })
    }
  })
}

// 使用
// 转化为 03-21 格式
<div class="time" v-format-time="'MM-DD'">
  {{ item.time }}
</div>
// 转化为  10小时前 格式
<div class="time" v-format-time.relative>
  {{ item.time }}
</div>
```

### defineExpose 使用

```ts
// 用户子组件向外暴漏自己的属性和方法，供父组件去使用

// 子组件
<script setup lang='ts'>
import { ref, defineExpose } from 'vue'

const num = ref<number>(0)
const handleUpdate = () => {
  num.value++
}

// 向外暴漏属性和方法
defineExpose({
  num,
  handleUpdate
})
</script>


// 父组件 接受属性和方法，并且使用
<template>
	<div id='app'>
  	<h2>num: {{ comRef?.num }}</h2>
		<button @click="comRef?.handleUpdate">num++</button>
  </div>
	<DomeVue ref='comRef' />
</template>
<script setup lang='ts'>
import { ref } from 'vue'

import DomeVue from './DomeVue.vue'

// 声明组件的ref对象
const comRef = ref<InstanceType<typeof DomeVue>>()

</script>
```

### Vue3 计算属性 set 和 get

```js
// 获取vuex的时候，有些时候页面刷新的时候，数据为undefined，但是由于vuex的数据又只能使用computed获取，因此可以在watch里面监听数据的变化，默认计算属性只有一个get方法，我们可以设置set方法，这样等数据获取到，可以下一步操作

const userInfo = computed({
  get: () => store.state.userInfo as UserInfoVuexTypes,
  set: () => {}
})

watch(
  () => userInfo.value,
  (newVal) => {
    userInfo.value = newVal
    // 设置用户消息数
    state.pageListData = userInfo.value.messageList ?? []
    //  当前分页显示的数据
    state.currentPageListData = state.pageListData.slice(0, 10)
    //  设置分页数量
    state.pageNum = Math.ceil(state.pageListData.length / 10)
  }
)
```

### watch 立即执行监听

```js
watch(
  () => route.path,
  () => {
    console.log(route)
  },
  {
    // 是否一刷新就立即执行监听
    immediate: true,
    // 是否深度监听复杂数据
    deep: true,
  }
)
```

### Props 设置参数类型和默认值

```js
// 在script setup中这种props类型和默认值

<script setup lang='ts' >
// defineProps  => props类型
// withDefaults => props默认值
import { defineProps, withDefaults } from 'vue'

interface PropsTypes {
  formItems: Array<string>
   // 设置是否必传
  labelWidth?: string
  info: { num: number }
}

// 声明props，并且赋默认值
const props = withDefaults(defineProps<PropsTypes>(), {
  labelWidth: '100px',
  formItems: () => ['vue', 'react'],
  info: () => ({ num: 22 })
})

</script>


// 页面展示时，不比带上前缀 props
<h2> labelWidth: {{ labelWidth }} </h2>

```

### 父组件调用子组件的方法

```vue
父组件获取子组件对象，并且调用子组件中的方法 // 父组件
<tempalte>
  <childVue ref='childRef'></childVue>
</tempalte>
<script lang="ts" setup>
import { ref } from 'vue'
import childVue from '@/components/childVue.vue'

// 声明子组件对象
const childRef = ref<InstanceType<typeof childVue>>()
// 调用子组件方法
childRef.value?.login()
</script>

// 子组件
<script lang="ts" setup>
import { ref, defineExpose } from 'vue'

// 子组件的方法
const login = (): void => {
  console.log('login方法调用')
}
// 将子组件的方法暴漏出去，供父组件调用
defineExpose({
  login,
})
</script>
```

### Vue3+Ts 使用动态组件

```vue
<template>
  <div class="component">
    <component :is="typeComponentMap[currIndex]"></component>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import componentA from './comonentA'
import componentB from './comonentB'

// 对应映射关系
const typeComponentMap = () => {
  0: componentA,
  1: componentB
}

// 当前组件索引
const currIndex = ref<number>(0)
</script>
```

### provide 与 inject 响应式传值

```js
// 通常情况下，我们要层级组件之间传值，会使用provide与inject，但是这个数据是静态的

// 静态传递
//  父组件
<script lang='ts' setup>
import { provide, readOnly } from 'vue'

const nums = ref<string>('数据')
// 发送数据 传递给子元素 通常使用readOnly 单向数据流，防止子组件修改父组件的数据
provide('nums', readOnly(nums.value))
</script>
// 子组件
<script lang='ts' setup>
import { inject } from 'vue'

// 接受数据
const nums = inject('nums')
</script>

// 动态传递 通过computed传递，这样就完成了响应式数据传递
//  父组件
<script lang='ts' setup>
import { provide, computed } from 'vue'

const nums = ref<string>('数据')
// 发送数据 传递给子元素
provide('nums', computed(() => nums.value))
</script>
// 子组件
<script lang='ts' setup>
import { inject, watch } from 'vue'

// 接受数据
const nums = inject('nums')

// 数据改变操作
watch(
    () => nums.value?.value,
    () => {
     	// 改变后的操作
    }
  )
</script>
```

### Suspense 异步加载组件

```vue
// 在项目中，如果一个子组件出错或者加载缓慢，对于用户来说体验极差，因此需要占位符，先用一些动画来代替展示，等到子组件加载完毕，再代替为了该子组件
<Suspense>
  <!-- 要加载的子组件 -->
  <template #default>
    <ArticleVue></ArticleVue>
  </template>
  <!-- 加载过程中的动画 -->
  <template #fallback>
		<!-- Element-plus中的加载动画 -->	
    <el-skeleton :rows="5" animated />
  </template>
</Suspense>
```

## 路由

### 动态加载路由

```js
// 在后台管理系统里面，我们根据不同的用户身份来获取不同的菜单，进而加载不同的组件

// 方案一：
// 注册全局的路由（写死），然后访问不同页面前，进行权限鉴别，看是否有权限

// 方案二：
// 动态注册服务器返回的路由 主要语法： router.addRoute()
// 痛点: 动态加载路由之后，页面刷新之后，会显示未加载该路由，使用本地存储，避免二次想服务器发送路由请求

// router.addRoute()
// 添加路由 path必须要带/
router.addRoute({
  path: '/admin',
  name: 'Admin',
  component: AdminVue
})
// 添加子路由
// 切记，第一个参数为父路由的name值，不是path值， 添加的子路由一定要加上父路径
router.addRoute('Admin', {
  path: '/admin/about',
  name: 'About',
  component: AboutVue
})

// utils/mapRouter.ts  路由映射表
// 这是是路由映射
// 这是是路由映射
import { RouteRecordRaw } from 'vue-router'

import HomeVue from '../views/Home.vue'
import AboutVue from '../views/About.vue'
import ArticleVue from '../views/Article.vue'
import LinkVue from '../views/Link.vue'
import MessageVue from '../views/Message.vue'
import AdminAdd from '../views/admin/Add.vue'
import AdminMange from '../views/admin/Mange.vue'

const allRoutes: RouteRecordRaw[] = [
  {
    path: '/admin',
    name: 'Home',
    component: HomeVue
  },
  {
    path: '/admin/about',
    name: 'About',
    component: AboutVue
  },
  {
    path: '/admin/article',
    name: 'Article',
    component: ArticleVue
  },
  {
    path: '/admin/link',
    name: 'Link',
    component: LinkVue
  },
  {
    path: '/admin/mes',
    name: 'Message',
    component: MessageVue
  },
  {
    path: '/admin/admintor/add',
    name: 'AdminAdd',
    component: AdminAdd
  },
  {
    path: '/admin/admintor/mange',
    name: 'AdminMange',
    component: AdminMange
  }
]

/**
 *
 * @param routes 用户菜单路由
 * @returns 该用户路由
 */
export default function (routes: any[]): RouteRecordRaw[] {
  const routers: RouteRecordRaw[] = []

  // 递归遍历 遍历二级路由
  const deep = (all: any[]) => {
    all?.forEach((item) => {
      if (item.children) {
        return deep(item.children)
      } else {
        const nowRouter = allRoutes.find(
          (router) => router.path === item.path
        ) as RouteRecordRaw
        routers.push(nowRouter)
      }
    })
  }

  deep(routes)

  return routers
}

// 在store中暴露一个方法， 一个用户刷新页面可以加载数据的方法
// 调用的是子模块的localLogin方法
export function localLogin() {
  store.dispatch('userInfo/localLogin')
}

// store/userInfo/index.ts
// 默认本地获取数据
actions: {
  async localLogin({ commit }) {
    // 先从本地获取存储的菜单信息 然后加载到vuex，然后动态加载路由
    const userInfo = cache.getItem('userInfo')
    commit('setUserInfo', { ...userInfo })
    const userMenu = cache.getItem('userMenu')
    commit('setUserMenu', userMenu)
    // @ts-ignore
    const routers = mapRouter(userMenu)
    // 切记，这是添加的父路由的name值，不是path，子路由必须要带上父路径
    routers.forEach((item) => router.addRoute('Admin', item))
  }
}

// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

import { localLogin } from '@/store'

// 页面刷新 重新加载路由
localLogin()

createApp(App).use(router).use(store).mount('#app')



// 后端路由规范
const role1 = [
  {
    path: "/admin",
    title: "首页",
  },
  {
    path: "/admin/mes",
    title: "消息管理",
  },
  {
    path: "/admin/about",
    title: "关于",
  },
  {
    path: "/admin/article",
    title: "文章管理",
  },
  {
    path: "/admin/link",
    title: "友情链接",
  },
  {
    path: "/admin/admintor",
    title: "管理员",
    children: [
      {
        path: "/admin/admintor/add",
        title: "添加管理员",
      },
      {
        path: "/admin/admintor/mange",
        title: "管理员管理",
      },
    ],
  },
]

```

### 封装页面跳转或刷新仍保留在当前位置上（Hooks）

```js
// 在我们平时开发网站中，运用最多的需求，就是用户在一个页面进入另一个页面，但用户返回的时候，希望继续回到刚才看的位置，而不是重新加载此页

// 首先我们需要在layout布局文件中配置<router-view />
// 利用作用域插槽，找到要显示的组件，并且将组件进行缓存
<router-view #default='scope'>
	<keep-alive>
  	<component :is='scope.Component'></component>
  </keep-alive>
</router-view>

// 封装一个scroll Hooks
// 位置：@/hools/scroll.ts
import { ref, onMounted, onDeactivated, onActivated } from 'vue'
// 前面封装的localStorage
import cache from '@/utils/cache'
import { useRoute } from 'vue-router'

// 传入当前位置的
export default function (){
  const scrollTop = ref<number>(0)
  const route = useRoute()
	const name = route.name as string
  // 记录当前浏览的位置
  onMounted(() => {
    window.addEventListener("scroll",() => {
      scrollTop.value = document.documentElement.scrollTop
    }, false)
  })

  // 离开时记录当前的位置
  onDeactivated(() => {
    cache.setCache(name, scrollTop.value)
  })

  // 进入时 重回当前位置
  onActivated(() => {
    const homeScroll = cache.getCache(name)
    window.scrollTo(0, homeScroll)
  })
}


// 使用
// 在article.vue中使用
<script setup lang='ts'>
import scroll from '@/hooks/scroll'

// 保留当前页面浏览进度
scroll()
</script>
```

### 路由导航守卫

```js
// 当我们做后台管理系统时， 用来判断用户是否已经登录，如果登录继续访问，没有登录则跳转到登录页，如访问 `/admin` ，如果没有登录，则跳转到 `/admin/login`

// 这里不再推荐使用next()
// 位置：router/index.ts
// 路由守卫
router.beforeEach((to: RouteLocationNormalized) => {
  const token = loaclCache.getCache('token')
  // 判断是否访问后台
  const reg1 = /^(\/admin)/
  // 访问后台
  if (reg1.test(to.path)) {
    // 未登录
    if (!token) {
      if (to.path !== '/admin/login') {
        return '/admin/login'
      }
    } else {
      if (to.path === '/admin/login') {
        router.go(-1)
      }
    }
  }
})
```

## pinia

### Vue3+ts 使用 pinia

```js
// 安装
npm install pinia -S

// 使用
// 目录
-- store
---- index.ts
---- userInfo.ts

// store/index.ts 统一导出文件
export { default as useUserInfoStore } from './userInfo'

// store/userInfo.ts
import { reactive } from 'vue'
import { defineStore } from 'pinia'

const useUserInfoStore = defineStore({
  const userInfo = reactive({
    name: ''
  })

	// 登录
	const login = async (name: string, pass: string) => {
    const { data } = await axios.post('/api')

    userInfo.name = data.name
  }

  // 统一导出属性和方法
  return { userInfo, login }
})

export default useUserInfoStore


// 使用
<script lang='ts' setup>
import { storeToRefs } from 'pinia'

import { useUserInfoStore } from '@/store'

const userInfoStore = useUserInfoStore()

// 拿去store里面的属性
const { userInfo } = storeToRefs(userInfoStore)
const { login } = userInfoStore
</script>
```

## vuex

### Vuex 分模块使用

```ts
// 在一个store中如果数据量过大，将难以维护，因为我们根据功能分不同的模块进行管理
// 结构
|- store
|--- index.ts (store的入口)
|--- types.ts (store中接口声明)
|--- login (登录模块)
|----- index.ts (登录模块入口)
|----- types.ts (登录模块接口)

// login/index.ts
import { Module } from 'vuex'
import type { LoginVuexData } from './types'
import type { StoreTypes } from '../types'

// 传入两个参数 （模块中的state类型，根store中的state类型）
const loginModule: Module<LoginVuexData, StoreTypes> = {
  // 模块的命名空间
  namespaced: true,
  state: {
    token: '',
    userInfo: {}
  },
  mutations: {
    // 更新token
    updateToken(state, { token }){
      state.token = token
    }
  },
  actions: {
    // 后台管理系统登录
    accountLogin({ commit }, loginData: any) {
      console.log('后台账户进行了登录', loginData)
    }
  },
  modules: {}
}

export default loginModule


// index.ts
import { createStore } from 'vuex'
import type { StoreTypes } from './types'
import loginStore from './login'

export default createStore<StoreTypes>({
  state: {
    name: '',
    age: ''
  },
  mutations: {},
  actions: {},
  modules: {
    loginStore
  }
})


// 在 .vue 文件中使用
<script lang='ts' setup>
import { reactive } from 'vue'
import { useStore } from 'vuex'
const adminLoginForm = reactive({
  name: '',
  pass: ''
})

const store = useStore()
// 调用actions
store.dispatch('loginStore/accountLogin', { ...adminLoginForm })
// 调用mutations
store.commit('loginStore/updateToken', { token })
</script>

```

### VueX 更好结合 TypeScript 使用

```js
// 当我们在组件中使用store时，如果需要获取某一个全局状态，但是没有任何的提示，因此我们可以封装使得Vuex更加兼容TypeScript

// 结构
|- store
|--- index.ts (store的入口)
|--- types.ts (store中接口声明)
|--- userInfo (用户信息)
|----- index.ts (用户信息)
|----- types.ts (用户信息接口)

// store/types.ts
// 导入userInfo模块接口类型
import { UserInfoTypes } from '../userInfo/types'
// 声明store根接口类型
export interface VuexRootTypes {
  author: string
  age: number
}

// 声明store模块接口类型
export interface VuexModuleTypes {
  userInfo: UserInfoTypes
  ....
}

// 交叉类型，将store根类型和store模块类型交叉
export type StoreTypes = VuexRootTypes & VuexModuleTypes

// store/index.ts
// 导入store 并且给useStore方法别名
import { createStore, Store, useStore as vuexUseStore } from 'vuex'
// 导入根store接口类型
import type { VuexRootTypes, StoreTypes } from './types'
import userInfo from './userInfo'

export default createStore<VuexRootTypes>({
  state: {
    author: '',
    age: 0
  },
  mutations: {
    setAuthor(state, { author, age }) {
      ;(state.author = author), (state.age = age)
    }
  },
  actions: {},
  modules: {
    userInfo
  }
})

export function useStore(): Store<StoreTypes> {
  return vuexUseStore()
}


// 使用
// App.vue
<script lang='ts' setup>
import { useStore } from '@/store'

// 这个时候就可以享用ts提示了
console.log(store.state.userInfo.name)

</script>

```

### Vuex 子模块调用根模块的 actions

```js
// 需求：在子模块actions调用根模块的actions中的方法

actions： {
  async actiongetInfos({ commit, dispatch }){
    // 调用本模块的mutations方法
    commit('updateInfo')
    // 调用本模块的actions方法
    dispatch('actionsUpdateInfo')
    // 调用根模块的actions方法
    dispathc('rootactions', null, { root: true })
  }
}
```

## 开发经验

### vue-cli5 最新版脚手架创建 vue3+ts+Eslint+prettier 项目报错解决方案

```js
//  第一个报错
Failed to load config "@vue/prettier" to extend from.

// 解决第一个报错
npm install @vue/eslint-config-prettier @vue/eslint-config-typescript -D

// 第二个报错
Failed to load config "@vue/prettier/@typescript-eslint" to extend from.
// 原因： 版本太新 降低版本

// 解决第二个报错
npm install @vue/eslint-config-prettier@6.* -D


```



### Vue3+ts 声明 json 文件

```js
// 在shims-vue.d.ts文件中

// 声明json文件
declare module '*.json'
```

### 不同环境下的配置

```js
// 在开发过程中，不同环境下的请求路径是不一样的，比如开发环境下和生产环境下

// 方法一：
// 利用process.env.NODE_ENV 来区分
// config.ts
let BASE_URL = ''
let BASE_NAME = ''
// 生产环境
if (process.env.NODE.ENV === 'development') {
  BASE_URL = 'http://loaclhost:3000/api'
  BASE_NAME = '开发阶段'
} else if (process.env.NODE.ENV === 'production') {
  BASE_URL = 'http://huangjianxin.cn/api'
  BASE_NAME = '生产环境'
} else {
  BASE_URL = 'http://local host:3000/test'
  BASE_NAME = '测试环境'
}

export { BASE_URL, BASE_NAME }

// 方法二：
// 分别在根目录上创建对应的.env文件

// 全局注入方法
BASE_URL
NODE_ENV
VUE_APP_随别写

// 开发环境
// .env.development
VUE_APP_BASE_URL = 'http://localhost:3000'
VUE_APP_BASE_NAME = '开发环境'

// 生产环境
// .env.production
VUE_APP_BASE_URL = 'http://huangjianxin.cn'
VUE_APP_BASE_NAME = '生产环境'

// 测试环境
// .env.test
VUE_APP_BASE_URL = 'http://localhost:test'
VUE_APP_BASE_NAME = '测试环境'

// 全局使用
// 在自定义变量前面加上 process.env
console.log(process.env.VUE_APP_BASE_NAME)
```

### Vue3 本地配置本地打包运行，非上线

```js
// 在vue.config.js中配置

module.exports = {
  // 打包目录
  outputDir: './build'
  // 打包之后的目录
  publicPath: './'
}
```

### vue3+Ts 中表单类型和组件类型

```js
// 基本用法
const target = ref<InstanceType<typeof '目标组件或者表单元素'>>()

// 在Element-plus中使用表单类型 比如获取表单 做表单的验证和校验
// 引入
import { ElForm } from 'element-plus'
// 声明表单
let Dataform = ref<InstanceType<typeof ElForm>>()

// 组件
import { LoginVue } from '@/views/client'
const LoginRef = ref<InstanceType<typeof LoginVue>>()

```

### 封装 LocalStorage

```js
// 封装浏览器的本地存储方法
// 在utils文件夹创建cache.ts
export enum CachaType {
  LocalStorage = 'local',
  SessionStorage = 'session'
}

// 本地缓存类
class LocalCache {
  setCache(
    key: string,
    value: any,
    localCacheType: CachaType = CachaType.LocalStorage
  ) {
    localCacheType === 'local'
      ? window.localStorage.setItem(key, JSON.stringify(value))
      : window.sessionStorage.setItem(key, JSON.stringify(value))
  }
  getCache(key: string, localCacheType: CachaType = CachaType.LocalStorage) {
    const text: any =
      localCacheType === 'local'
        ? window.localStorage.getItem(key)
        : window.sessionStorage.getItem(key)
    return text ? (text =  (text)) : text
  }
  removeCache(key: string, localCacheType: CachaType = CachaType.LocalStorage) {
    localCacheType === 'local'
      ? window.localStorage.removeItem(key)
      : window.sessionStorage.removeItem(key)
  }
  clearCache(localCacheType: CachaType = CachaType.LocalStorage) {
    localCacheType === 'local'
      ? window.localStorage.clear()
      : window.sessionStorage.clear()
  }
}

export default new LocalCache()


// 使用
import localCache, { CachaType } from '@/utils/cache'
// 存储
localCache.setCache('token', "xinxin", CachaType.LocalStorage)
// 获取
localCache.getCache('token', CacheType.LocalStorage)
// 删除
localCache.removeCache('token')
// 清空
localCache.clear()

```

### 使用顶部加载条

```js
// 安装
npm install nprogress --save && npm install @types/nprogress --save-dev

// 用来路由切换 显示进度条  router.ts
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

//进度条配置
NProgress.configure({
  easing: 'ease', // 动画方式
  speed: 500, // 递增进度条的速度
  showSpinner: true, // 是否显示加载ico
  trickleSpeed: 200, // 自动递增间隔
  minimum: 0.3 // 初始化时的最小百分比
})

router.beforeEach((to, from, next) => {
  // 每次切换页面时，调用进度条
  NProgress.start()
  next()
})

router.afterEach((to, from) => {
  // 在即将进入新的页面组件前，关闭掉进度条
  NProgress.done()
})

```


### dayjs 使用

```js
// 超级好用的时间转化库
// 安装
npm install dayjs --save

// 用法
dayjs(1643077195000).format('YYYY-MM-DD HH:mm:ss')

// 属性
YYYY  => 年， 2022
YY    => 年， 22
M     => 月， 1-12
MM    => 月， 01-12
D     => 日， 1-31
DD    => 日， 01-31
H     => 时， 0-23
HH    => 时， 00-23
h     => 时， 1-12
hh    => 时， 01-12
m     => 分， 0-59
mm    => 分， 00-59
s     => 秒， 0-59
ss    => 秒， 00-59


// 二次封装 在utils下新建date-time.ts
// 这里是时间转化代码
import * as dayjs from 'dayjs'
// 中文语言
import 'dayjs/locale/zh-cn'
// 使用过去时间转化插件relativeTime
import rTime from 'dayjs/plugin/relativeTime'
// 使用utc格式
import utc from 'dayjs/plugin/utc'

// 使用中文语言包。固定格式
dayjs.locale('zh-cn')
// 使用插件。固定格式dayjs.extend(插件)
dayjs.extend(rTime)
// 使用插件 支持utc格式
dayjs.extend(utc)

// 相对时间  传入过去的时间戳 返回过去多久
/**
 *
 * @param num utc和时间戳格式
 * @returns 过去多久
 */
export const relativeTime = (num: number | string): string => {
  // @ts-ignore
  return dayjs().to(dayjs(num))
}

// utc格式使用
/**
 *
 * @param utc utc时间(string)
 * @param format 时间格式(YYYY-MM-DD HH:mm:ss)
 * @returns 时间格式(format)
 */
export const dateFromatUTC = (
  utc: string,
  format = 'YYYY-MM-DD HH:mm:ss'
): string => {
  return dayjs.utc(utc).format(format)
}

// 时间戳格式使用
/**
 *
 * @param stamp 时间戳(number)
 * @param format 时间格式(YYYY-MM-DD HH:mm:ss)
 * @returns 时间格式(format)
 */
export const dateFromatStamp = (
  stamp: number,
  format = 'YYYY-MM-DD HH:mm:ss'
): string => {
  if (stamp.toString().length === 10) stamp = stamp * 1000
  return dayjs.utc(stamp).format(format)
}



// 使用
// Home.vue
<script lang='ts' setup>
import dayjs, { relativeTime } from '@/utils/date-time'

// 2022-01-25 10:19:55
console.log(dayjs(1643077195000).format('YYYY-MM-DD HH:mm:ss'))
</script>

```

### vue3+Ts 引入 element-plus

```js
// 方式一：推荐
// 安装
npm install element-plus —-save
// 全局引入
import ElementPlus from 'element-plus'
import 'element-plus/theme-chalk/index.css'
createApp(App).use(router).use(store).use(ElementPlus).mount('#app')


// 方式二：
// 安装
vue add element-plus

// 在mian.ts修改
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

import ElementPlus from 'element-plus';
import 'element-plus/lib/theme-chalk/index.css';

const app = createApp(App)
app.use(store).use(router).use(ElementPlus).mount('#app')


// 按需引入Element-plus

```

### Vue3+Ts 引入 Animate.css

```js
// 安装
npm install animate.css -S

// mian.ts中声明
import "animate.css"

// 使用
.animate{
  // 动画名称
  animation: fadeInUp;
  // 动画持续时间
  animation-duration: 1s;
}
// 目前地址： https://www.dowebok.com/demo/2014/98/

// js中动态添加
const welfareRef = ref<InstanceType<typeof Element>>()

onMounted(() => {
  document.addEventListener("scroll", () => {
    if (document.documentElement.scrollTop > 500) {
        welfareRef.value?.classList.add('animate__animated')
        welfareRef.value?.classList.add('animate__fadeInUp')
      }
  }, false)
})
```

### vue3+Ts 使用 echarts

```vue
// 安装 npm install echarts -S // 使用
<template>
  <div ref="echartsRef"></div>
</template>
<script lang="ts" setup>
const { ref, onMounted } from 'vue'
const * as echarts from 'echarts'

const echartsRef = ref<HTMLElement>()
onMounted(() => {
  // 类型断言，确定有值
  const myChart = echarts.init(echartsRef.value!)
  const option = {
    title: {
      text: 'ECharts 入门示例'
    },
    tooltip: {},
    legend: {
      data: ['销量']
    },
    xAxis: {
      data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
    },
    yAxis: {},
    series: [
      {
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      }
    ]
  }
  // 使用echarts
  myChart.setOption(option)
})
</script>
```

### vue3+TS 对 axios 二次封装

```js
//axios二次封装
import axios, { AxiosRequestConfig, AxiosResponse } from 'axios'

// 使用loading加载
import { ElLoading } from 'element-plus'

let loading: any

interface Options {
  lock: boolean;
  text: string;
  background: string;
}

// 显示加载动画
const startLoading = (): void => {
  const options: Options = {
    lock: true,
    text: '正在加载中...',
    background: 'rgba(255,255,255,.7)',
  }
  loading = ElLoading.service(options)
}

// 关闭加载动画
const endLoading = (): void => {
  loading.close()
}

axios.defaults.baseURL = `http://localhost:3000`
// axios.defaults.withCredentials = true

// 添加请求拦截器
// 在发送请求之前做些什么
axios.interceptors.request.use((config: AxiosRequestConfig) => {
  // 开启loading加载动画
  startLoading()
  let token = localStorage.getItem('token')
  if (!config.headers) return
  // 添加token
  config.headers.Authorization = 'Bearer ' + token
  return config
})
// 添加响应拦截器
axios.interceptors.response.use(
  (response: AxiosResponse<any>) => {
    // 对响应数据做点什么
    // 关闭loading加载动画
    endLoading()
    return response
  },
  err => {
    // 对响应错误做点什么
    // 关闭loading加载动画
    endLoading()
    return Promise.reject(err)
  }
)

export default axios
```

### vue 引入 echarts 图表

```js
// 安装
npm i echarts -S

// 在需要的页面引入
import * as echarts from 'echarts'

// 然后在页面上创建一个容器
<div  id="myChart"></div>

// 使用
<script lang='ts' setup>
import * as echarts from 'echarts';
import { onMounted } from 'vue'
onMounted(() => {
  type EChartsOption = echarts.EChartsOption

  var chartDom = document.getElementById('myChart')!
  var myChart = echarts.init(chartDom)
  var option: EChartsOption

  option = {
    xAxis: {
      type: 'category',
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
      type: 'value'
    },
    series: [
      {
        data: [820, 932, 901, 934, 1290, 1330, 1320],
        type: 'line',
        smooth: true
      }
    ]
  }
  // 绘制
  option && myChart.setOption(option)

  // 浏览器窗口改变时，图表跟着变化
  window.addEventListener(
  	'resize',
    () => {
      myChart.resize()
    }
  )
})

</script>
```

### vue 项目上线处理

```js
// 配置服务器路径修改 因为本地开发和线上版本服务器地址是不一样的
// 根据运行环境决定是生产版本还是线上版本

const baseURL = process.env.NODE_ENV === "development"?"http://localhost:3000":""


// 运行打包命令
npm run build

// 此使会多出一个dist文件夹，然后把里面的所有内容复制到后端的public中，然后直接运行后端 浏览器输入后端的地址 即可看到前端打包后的页面

// 打包之后 如果我们直接从浏览器地址框进行跳转调式，发现会出现一个后端无此路由的页面，这是因为我们现在是基于后端进行调式，所以浏览器输入网址优先后端，这时我们需要做一个改变

// 在serve文件夹下的index.js加入下面代码
// 让路由正常跳回前端路由

const path = require("path")
// 放到最后 意思前端的路由都解决不了的话 通过这里解决 然后直接导入到index.html 文件
app.use("*",(req,res)=>{ 					res.sendFile(path.join(__dirname,"./static/index.html"))
})

```

### vue3+TS 引入 md 编译器和解析器

```js
// 安装
npm install md-editor-v3 --save

// 引入编译器
<template>
  <div id="about">
    <md-editor v-model="text" style="height: 100vh;" />
  </div>
 </template>

 <script lang="ts">
 import { defineComponent,ref } from 'vue';
 import MdEditor from 'md-editor-v3';
 import 'md-editor-v3/lib/style.css';

 export default defineComponent({
   components: { MdEditor },
   setup(){
     let text = ref<string>("")

     return {
       text
     }
   }

 });
 </script>
 <style scoped lang="less">
 #about{
   width: 100%;
 }
 </style>


// 引入解析器
<template>
  <div id="about">
    <md-editor
    v-model="text"
    previewOnly
  />
  </div>
 </template>

 <script lang="ts">
 import { defineComponent,ref } from 'vue';
 import MdEditor from 'md-editor-v3';
 import 'md-editor-v3/lib/style.css';

 export default defineComponent({
   components: { MdEditor },
   setup(){
     let text = ref<string>("")

     return {
       text
     }
   }

 });
 </script>
 <style scoped lang="less">
 #about{
   width: 100%;
 }
 </style>



```

### vue3 引入 md 编译器和解析器

```js
// 安装编译器
npm i @kangc/v-md-editor -S
// ts
@types/kangc__v-md-editor

// 在main.js引入
import VueMarkdownEditor from '@kangc/v-md-editor';
import '@kangc/v-md-editor/lib/style/base-editor.css';
import vuepressTheme from '@kangc/v-md-editor/lib/theme/vuepress.js';
import '@kangc/v-md-editor/lib/theme/style/vuepress.css';
import Prism from 'prismjs';
import VMdPreview from '@kangc/v-md-editor/lib/preview';
import '@kangc/v-md-editor/lib/style/preview.css';
import githubTheme from '@kangc/v-md-editor/lib/theme/github.js';
import '@kangc/v-md-editor/lib/theme/style/github.css';
import hljs from 'highlight.js';


VueMarkdownEditor.use(vuepressTheme, {
  Prism,
});

VMdPreview.use(githubTheme, {
  Hljs: hljs,
});


VueMarkdownEditor.use(vuepressTheme);

const app = createApp(App)

app.use(store).use(router).use(VMdPreview).use(VueMarkdownEditor).mount('#app')


// 在需要的页面写入 编译器
<template>
  <v-md-editor v-model="text" height="800px"></v-md-editor>
</template>
<script>
export default {
  name: "markDown",
  methods:{
  },
  data() {
    return {
      text: '',
    };
  },
};
</script>
<style scoped>
</style>

// 在需要的页面引入 解析器
<template>
 <v-md-preview :text="text"></v-md-preview>
</template>
<script>
export default {
  name: "markDown",
  methods:{
  },
  data() {
    return {
      text: '',
    };
  },
};
</script>
<style scoped>
</style>
```

### vue3+Ts 引入富文本编辑器

````js
// 安装
npm install wangeditor -S

// 新建editor.vue组件
<template>
  <div>
    <div className="shop">
      <div className="text-area">
        <div
          ref="editorElemMenu"
          style="backgroundcolor: '#f1f1f1'; border: '1px solid #ccc'"
          className="editorElem-menu"
        ></div>
        <div
          style="height: 300;border: '1px solid #ccc',borderTop: 'none'"
          ref="editorElemBody"
          className="editorElem-body"
        ></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { onMounted, reactive, toRefs, defineComponent, ref } from 'vue'
import E from 'wangeditor'

interface DataProps {
  editor: any
  editorContent: string
  getContent: (ref?: any) => void
}

export default defineComponent({
  name: 'Editor',
  components: {},
  props: {
    options: Object,
    value: String
  },
  setup(props, { emit }) {
    const editorElemMenu = ref()
    const editorElemBody = ref()
    const data: DataProps = reactive({
      editorContent: '',
      editor: {},
      getContent: () => {
        console.log('111', props.value)
        data.editor.txt.html(props.value)
      }
    })
    onMounted(() => {
      const elemMenu = editorElemMenu.value
      const elemBody = editorElemBody.value
      data.editor = new E(elemMenu, elemBody)
      // 使用 onchange 函数监听内容的变化，并实时更新到 state 中
      data.editor.config.onchange = (html: any) => {
        // console.log(data.editor.txt.html())
        data.editorContent = data.editor.txt.html()
        // 向外部返回一个处理过的值
        emit('onEditor', data.editor.txt.html())
        emit('update:value', data.editor.txt.html())
      }
      data.editor.config.customUploadImg = function (files: any, insert: any) {
        // files 是 input 中选中的文件列表insert 是获取图片 url 后，插入到编辑器的方法
        // let file;
        // if (files && files.length) {
        //     file = files[0];
        // } else {
        //     return
        // }
        // 图片上传
        console.log('files1', files)

        const formData = new FormData()
        formData.append('file', files[0])
        console.log('files', files, insert)
      }

      data.editor.config.menus = [
        'head', // 标题
        'bold', // 粗体
        'fontSize', // 字号
        'fontName', // 字体
        'italic', // 斜体
        'underline', // 下划线
        'strikeThrough', // 删除线
        'foreColor', // 文字颜色
        'backColor', // 背景颜色
        'link', // 插入链接
        'list', // 列表
        'justify', // 对齐方式
        'quote', // 引用
        'emoticon', // 表情
        'image', // 插入图片
        'table', // 表格
        'video', // 插入视频
        'code', // 插入代码
        'undo', // 撤销
        'redo' // 重复
      ]
      data.editor.config.uploadImgShowBase64 = true
      data.editor.create()
    })
    const refData = toRefs(data)
    return {
      ...refData,
      editorElemMenu,
      editorElemBody
    }
  }
})
</script>

<style scoped>
.editorElem-menu {
  background-color: red;
  border: 1px solid rgb(199, 195, 195);
}
.editorElem-body {
  text-align: left;
}
.part_right {
  width: 100%;
  background: #f2f2f2;
  min-height: 100vh;
}
.list {
  width: 96%;
  margin: 0 auto;
  padding-top: 50px;
}
.list ul li {
  list-style-type: none;
  display: flex;
  border-bottom: 1px solid #e6e5e5;
  min-height: 50px;
  background: #d5d5d5;
}
.list ul li > div {
  flex: 1;
  line-height: 50px;
}
.list ol li {
  list-style-type: none;
  display: flex;
  border-bottom: 1px solid #e6e5e5;
  min-height: 30px;
}
.list ol li > div {
  flex: 1;
  line-height: 30px;
}
.flright {
  float: right;
  margin-right: 2%;
}
</style>
  
  
  
// 使用该组件
<template>
  <div id="about">
    <button @click="handleClick">获取富文本中的内容</button>
  <EditorVue ref="Editor" :value="data.content" @onEditor="data.onEditor" />
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive, onMounted } from 'vue'
import EditorVue from './editor.vue'

const textData =
  '<h1 id="w0e7l">Vue3响应式原理</h1><p data-we-empty-p="" style="text-align:left;">&nbsp; &nbsp; &nbsp; &nbsp; 我所向往的世间，哦，还有时间，就是在明天<br/></p><p data-we-empty-p="" style="text-align:left;"><strike>123</strike><br/></p><p data-we-empty-p="" style="text-align:left;"><strike>123123123123123</strike><span style="text-align: center;">12312312<font color="#46acc8">31231231 1231231231231 1</font><font color="#000000">23123123123</font></span></p><ul><li style="text-align:left;"><span style="text-align: center;"><font color="#000000">大手大脚案例</font></span></li><li style="text-align: center;"><font color="#000000">爱上你端是丢啊</font></li><li style="text-align: center;"><font color="#000000">案涉及到司机第哦啊设计的</font></li></ul><div style="text-align:left;"><br/></div>'

const Editor = ref()
const data = reactive({
  content: '1',
  //获取富文本中的内容
  onEditor: (value: string) => {
    console.log('父组件内容：', value)
  },
  // 富文本回显
  showBack: () => {
    // data.content = '回显内容公众号qdleader'
    setTimeout(() => {
      Editor.value.getContent()
    })
  }
})

// 模拟修改文章 先获取旧的文章
data.content = textData

const handleClick = () => {
  // 获取文章的简介 前125个字符
  let value = Editor.value.editorElemBody.innerText.slice(0, 125)
  // 去除换行
  function ClearBr(key: string) {
    key = key.replace(/<\/?.+?>/g, '')
    key = key.replace(/[\r\n]/g, '')
    return key
  }
  // 去除空格
  function CTim(str: string) {
    return str.replace(/\s/g, '')
  }
  value = ClearBr(value)
  value = CTim(value)
  console.log(value)
}

onMounted(() => {
  data.showBack()
})
</script>

<style scoped lang="less"></style>

  
// 展示富文本内容
只需要在vue标签中通过 v-html 绑定即可，因为富文本都是各种样式的元素

````
