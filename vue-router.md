Vue-Router

[TOC]

##### 基础

######  一、普通、动态路由传参方式

```javascript
// 路由代码传参
import About from 'about'
// routes 配置
{
  path: '/about/:id', // 动态路由
  component: About,
  props: true // ①布尔模式
}

{
  path: '/about', // 普通路由
  component: 'About',
  props: { id: 19 } // ②对象模式
}

// 接收方式 props
props;['id'] 或者
props: {
  id: { type: Number, default: 12}
}
// ③函数模式
routes:[
  {
    path: '/about',
    component: About,
    // props: route => ({id:route.query.id}) // url='/about?id="89"' 或者
    props: route => ({id: route.params.id}) // url='/about/:id' => '/about/89'
  }
]
```

###### 二、动态路由：

* 将给定匹配模式的路由映射到同一个组件，复用一个组件，相对与销毁后重建更高效。

* Keep-alive包括时，组件的声明周期钩子函数不会被重复调用。

* 要对同一个组件中参数变化做出响应的话，可以通过watch 侦听$route对象上的任意属性

	```javascript
	watch: {
	  $route: {
	    immediate: true,
	    handler(route) {
	      // 处理事件 对路由变化做出响应
	    }
	  }
	}
	```

* 或者使用导航守卫，beforeRouteUpdate，也可以取消导航

###### 三、捕获所有路由或404路由

###### 四、路由的匹配语法

* 自定义正则 像可以区分 /list/100 和/list/xsk 等路由
	*  routes: [ { path: '/list/:id(\\\\d+)'},  {path: '/list/:name'} ]
* 可以重复的参数 匹配多个部分的路由，可以用 \* 号和 +号将参数标记为重复

* 也可通过使用?号修饰符（0个或1个）将一个参数标记为可选

###### 五、嵌套路由、命名路由

###### 六、编程式导航

* 声明式（<router-link :to="..."></router-link>）\编程式路由 router.push(...)

* Router.push(params): 

	* Params: 字符串路径、路径对象、命名的路由并加上参数、带查询参数、带hash

	* ```javascript
		'/users/detail'
		{ path: '/users/detail' }
		{ name: 'detail', params: {id: '0'}}
		{ path: '/users/detail', query: {id: '0'} }
		```

* 替换当前位置 router.replace({path: '/users'}) 或者router.push({path:'users', replace: true}); 导航时不会向history添加记录
* history.go()横跨历史

###### 七、命名视图： <router-view name="sideBar"></router-view>

###### 八、重定向配置 

```javascript
// 通过routes配置来完成
const routes = [{ path: '/home', redirect: '/'}]
// 重定向的目标也可以是一个命名的路由  redirect: { name: 'Details'}
// 一个方法动态返回重定向目标
const routes = [
  {
    path: '/home/:id',
    redirect: to => {
      return {path:'Details', query: { q: to.params.searchText}}
    }
  }
]
// 别名
alias: '/home'
```
###### 九、路由组件传参 props、$route.query\$route.params

* 布尔模式 routes配置时 props:true设置即可

* 对象模式 props: { id: '0' } 当props为静态的时候很有用

* 函数模式 创建一个返回props的函数，允许你将参数转换为其他类型，将静态值与基于路由的值相结合等操作

	```javascript
	props: route => ({ query: route.query.id })
	props: route => ({ params: route.params.id})
	```

* 对于命名视图的路由，必须为每个命名视图定义props配置

	```javascript
	const routes = [{
	  path: '/home',
	  components: { default: Home, sidebar: Sidebar},
	  props: { default: true, sidebar: true}
	}]
	```

###### 十、不同的历史模式

* Hash模式：history:  createWebHashHistory()    SEO受影响
* HTML5模式：history：createWbeHistory()       如果没有适当的服务器配置，就会404，需要在服务器上添加一个简单的回退路由



##### 进阶

###### 一、导航守卫：

* vue-router提供的主要是通过跳转或取消的方式守卫导航。

* 方式：全局、单个路由独享、组件

* 全局前置守卫：beforeEnter:
	* 每个守卫都接收两个__参数__：to\from\next(可选)
	* __返回值__ ①false:取消当前导航、②一个路由地址：通过一个路由地址跳转到不同的地址，类似于router.push()可配置，当前的 导航被中断然后进行一个新的导航。
	* next可选参数

* 全局后置守卫：afterEach 不接受next函数也不会改变导航本身
* 全局解析守卫：beforeResolve

* 路由独享守卫：在routes中配置

* 组件内的守卫   可用配置API：beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave

	* beforeRouteEnter:唯一可传递next回调守卫；解决不可访问this；

	* next（）里的内容执行时机在组件mounted周期之前；
	* beforeRouteUpdate: 该组件复用时被调用

![完整的导航解析流程](/Users/xsk-walter/EastHope/Vue/vue-router/完整的导航解析流程.png)

###### 二、路由元信息：配置meta字段

* 在routes配置中添加meta属性，值是对象形式；
* 路由匹配的所有路由记录会暴露为$route对象中，$route.meta方法，包含所有的meta字段；

###### 三、过渡动效  V3.x版本

```vue
// ①过渡模式 out-in \ in-out
<transition mode="out-in">
  <router-view></router-view>
</transition>

//style中设置 - 根据需求设置
<style>
.v-enter { // 进入过渡开始状态
    opacity: 0;
}

.v-enter-active {// 进入活动状态
    transition: 0.5s;
}

.v-enter-to { // 进入的结束状态
    opacity: 1;
}

.v-leave { // 离开过渡的开始状态
    opacity: 1;
}

.v-leave-active {// 离开活动状态
    transition: 0.5s;
}
  
.v-leave-to {} // 离开结束状态
</style>

// ②动态设置name 实现过渡效果
<transition :name="name">
  <router-view></router-view>
</transition>

<script>
	export default {
    data() {
      return { name: 'fade' }
    }
  }
</script>

<style>
// name + 'enter-active' 组合来设置各个状态的动画效果
.fade-enter-active {
  transition: all 0.3s ease-out;
}

.fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}

.fade-enter-from,
.fade-leave-to {
  transform: translateX(20px);
  opacity: 0;
}
</style>
```

###### 四、滚动行为

* 注：只在支持history.pushState的浏览器可用
* v3.x => v4.x ; v4.x 用 （top、left） 替换 （y、x）表示距离
* vueRouter实例化时，提供一个scrollBehavior方法，（to, from,savePosition）,其中savePosition只在浏览器前进后退时才有效

###### 五、路由懒加载、动态路由
