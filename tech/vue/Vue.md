## VueJS

>Vue.js (读音 /vjuː/，类似于 view) 是一套构建用户界面的渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，它不仅易于上手，还便于与第三方库或既有项目整合。

<img src=".\lifecycle.png" style="width:70%; height:50%;" alt="图片名称" align=center />

[TOC]

### Vue知识点一览

#### 构建说明

<img src=".\clipboard.png" style="width:80%; height:100%;" alt="图片名称" align=left />

#### template - 模板 

vue的模板可分为： html模板，单组件模板（推荐），js内联模板（vue实例使用），x-template，组件的内联模板（弃用）

模板智能有一个根节点

##### 指令

- v-bind(:)[<属性>] : 不指定属性可绑定一个对象 
- v-on(@)<事件>:  html原生事件/组件自定义事件
  - 事件修饰符
    - stop: 停止冒泡
    - prevent: 阻止默认行为
    - capture: 捕获阶段
    - self: 只在元素本身
    - once: 只执行一次
  - 键盘修饰符
    - @keyup.enter  / .13 / ctrl / page-down
  - 鼠标修饰符
    - @click.left / right /middle
- v-if: 操作dom, 没有key属性会重用dom， 惰性的，支持template
- v-show: display:none，不支持template
- v-for: 
  - 内容格式：(val, key, index) of items
  - key是必须的
  - 和v-if一起使用时优先级高于v-if，
- v-model: 双向绑定，相当于:value+@input事件
  - .trim
  - .number: 转为number
  - .lazy: input change时同步
- v-html
- 自定义指令

##### 插值 - {{ }}

- 模板表达式都被放在沙盒中，只能访问全局变量的一个白名单，如 `Math` 和 `Date` 。你不应该在模板表达式中试图访问用户定义的全局变量。

#### data - 状态

​	只有在初始化时定义的属性才是响应式的

#### methods - 方法

​	template中使用的方法才需要定义到methods下

#### computed - 计算属性

​	 保证数据的source-of-truth，能够通过现有属性计算出的属性，可以放到计算属性中。计算属性会基于依赖缓存

#### components - 局部注册组件

​	参照组件章节

#### watch - 属性监听

​	监控data属性的变化并执行回调函数

#### 生命周期钩子

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy


- 内置方法
  - vm.$refs：不要在template中和计算属性中使用
  - vm.$emit
  - vm.$on



### 工具

vue-cli 命令行工具

```bash
$ npm install -g vue-cli
$ vue init webpack <project>
```



Vue Devtools (需要翻墙)

https://github.com/vuejs/vue-devtools#vue-devtools





### 组件

组件的定义主要包含：属性、事件和插槽

`局部注册的组件如果为递归调用的需要定义name属性`

一些HTML元素只有在特定的元素内才是有效的如：ul > li, 为了避免错误应写成`<li is="my-li">`

组件的data属性需要写成函数形式

#### 注册

```javascript
// 全局
Vue.component('MyComponent', MyComponent);
// 全局 异步(webpack)
Vue.component('MyComponent', () => import('@/components/MyComponent'));
// 局部注册 -- vue实例/组件的配置对象中添加
components: {
  MyComponent,
}

```

#### 命名

```vue
// template中使用 kebab（大串）命名法
// 组件声明中组件名使用PascalCase（帕斯卡）命名法
// 组件声明中组件名属性名使用camelCase（驼峰）命名法

// template中使用命名和JS
```



#### 属性

*组件中不允许修改属性值*

```vue
<template>
  	<!--父组件 -->
	<!--template中使用属性-->
  	<!--使用:counter, 传入的值为number类型，否则为string类型-->
	<my-component my-class="comp" :counter="1"> </my-component>
	<!--未定义的属性会保留到子组件的根节点，重复的属性会合并-->
	<my-component not-defined class="pclass"> </my-component>
</template>

<script>
// 属性的声明
Vue.component('MyComponent', {
  props: ['firstAtt', 'myClass'],
  /* 校验和默认值
  props: {
    counter: Number,
    default: 0,
    validator: function (value) {
      return value > 0;
    }
  },
  */
  mounted() {
    // 组件中使用属性
    console.log(this.counter);
  }
});
</script>


```

#### 事件

子组件使用this.$emit触发事件，父组件使用v-on/@监听事件

```vue
<template>
 	<!--父组件 -->
	<counter :init="0" @increase=""> </counter>
  	<!--双向绑定 -->
  	<counter :foo.sync="bar"> </counter>
  	<!--相当于 -->
  	<counter :foo.sync="bar" @update:foo="val => bar = val"> </counter>
    
  	<!--v-model -->
  	<counter v-model="foo"> </counter>
  	<!--相当于 -->
   	<counter v-model="foo" @change="val => foo = v"> </counter>
  
</template>

<script>
// 属性的声明
Vue.component('counter', {
  model: {
    value: 'v', // 默认为 value
    event：'change', // 默认为 input
  }
  props: ['init'],
  data() {
    return {
  	  v: '',
      val: this.init,
    };
  },
  methods: {
    increase() {
      this.val++;
      this.$emit('increase', );
      // this.$emit('update:foo', newValue); 
    },
  },
});
</script>


```



#### 插槽

```vue

<!--父组件 -->
<counter v-model="foo">
	<span> 默认插槽 </span>
  	<span slot="s1"> 命名插槽 </span>
 	
  	<!--作用域：template必须存在 -->
    <template slot-scope="props" slot="ws">
      <span>hello from parent</span>
      <span>{{ props.text }}</span> <!--666 -->
    </template>
</counter>

<!--子组件 -->
<div>
	<slot> 默认插槽 </slot> <!--默认值 -->
  	<slot name="s1"> 命名插槽 </slot>
  	<slot name="ws" text="666"> 作用域 </slot>
</div>

<!--动态组件 -->
<component is=""></component>
```



### 动画

### vue-router

```vue
<template>
	<router-view>
     <router-link to="{name:'xxx', path:'xxx' params:'xxx', query:'xxx'}">go</router-link>
</template>
<script>
/* 安装 */
// npm install vue-router -S

import VueRouter from 'vue-router';

export default new VueRouter({
  	mode: 'hash' // 可选, history|hash
	// 路由表
	routes: [{ // 路由项配置
      name: '', // 命名路由， 可选
      path: '',
      redirect: '' // 重定向，和旋
      alias: '', // 别名
      component: // ()=>import('path/to/component') 异步路由组件，component：同步组件
  	  components: {	// 有多个<router-view>标签时要使用components，name为router-view的name
  		default: compoent // 没有name的router-view
     	name: compoent 
  	  },
      children: [{
        // 子路由信息
      }]
      beforeEnter（） {
        
      } 
	}]
});

// vue的router选项
new Vue({
  router,
})

// 生命周期，导航守卫
router.beforeEach(function(from, to, next){
  // 必须调用next
})

vm.beforeRouteLeave
router.beforeEach
vm.beforeRouteUpdate // -- 参数，query watch $route
route.beforeEnter // 路由项配置
vm.beforeRouteEnter // -- 取不到this，在next回调函数
router.beforeResolve
router.afterEach


// 组件注入
// from, to...
this.$router // push, replace, go, back, forward, matched, fullpath....
this.$route // 当前的路由信息对象
</script>
```



- 子路由不能为命名路由

- 嵌套路由，父路由的path为/parent/:id时如果匹配不到子路由，则可以定义一个path为' '的路由作为默认显示

- 查询功能的参数，查询定义到url中，用户保存/刷新页面时可以保持状态

- uri采用restful设计，可以定义类型：比如ID为数字型才匹配

- 目录名、index文件全小写

- 组件文件、类首文件字母大写，代码中的变量首字母大写

- 动态的首页可以在动态路由中定义alias为/

- path+params不兼容，和query兼容

- 参数或查询的改变并不会触发进入/离开的导航守卫

- 守卫中要调用next否则会阻塞之后的执行--加载动态路由

- 复杂页面可以使用keep-alive缓存

- router.mode = 'history', History模式，需要服务器全局返回index.html

- 额外的路由信息对象的meta，在$route.matched中可以访问

- 响应快（3s内）在导航前取数据（+NProgress显示进度）+ 响慢在组件声明周期取数据

- 尽量使用path定义和跳转路由

  ​



路由的使用场景和定义？

- 静态的路由登录、注册、密码找回、404、401路由，info页【实例化router的时候配置到路由表】
- 动态权限路由加载后使用router.addRoutes动态加载
  - 服务器传值：path，组件名（前端根据组件名获取组件），排序，图标，是否隐藏（显示到菜单）
  - 动态的首页：动态的首页路由项配置alias: '/'
- 带参数和查询条件的路由
  - 参数和查询条件定义到url上（而不是保存在组件中，用户保存连接时查询可用，api通用）
  - 组件会被重用，要watch $route或者beforeRouteUpdate中做数据获取。




### 常见问题

只有当实例被创建时 data 中存在的属性是响应式的

数组对象更改的检测

Vue 实例暴露了一些有用的实例属性与方法。它们都有前缀 $

不要在选项属性或回调上使用箭头函数

vue-template中不能有script标签

### 资源

教程

api

awesome

vue-element-admin

源码