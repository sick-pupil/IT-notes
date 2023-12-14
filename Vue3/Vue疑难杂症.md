## 1. el、template、render、mount
- 编译：从模板字符串生成`Dom`，如`<div>template</div>`只是一个单纯的模板字符串，只有在`createElement('<div>template</div>')`后才生成真正的`Dom`
- 挂载元素：`Vue`生成的`Dom`覆盖在该挂载元素上，即`Vue`生成的`Dom`替换该挂载元素

### 1. el
提供一个在页面中已存在的`Dom`元素作为`Vue`实例的挂载元素，如果`render`与`template`属性都不存在，则被`el`指明的挂载`Vue`实例的`Dom`元素`HTML`，会被当作模板
```js
new Vue({
	el: '#app'
})
```
### 2. template
指定一个字符串模板，模板将会替换被挂载的元素，被挂载元素的内容会被替换；实际项目挂载模板常用`App.vue`
```js
new Vue({
	template: "<div> {{ msg }} </div>"
})
```
### 3. render
不使用`template`模板替换挂载元素，接收`createElement`方法创建`VNode`
```js
import App from './App'

new Vue({
	render(createElement) {
		return createElement(App)
	}
})

/**
new Vue({
	el: '#root',
	template: '<App></App>',
	components: {
		App
	}
})
**/
```
### 4. mount
与`el`不同，`el`是在实例化时指定挂载目标，底层会自动挂载；如果没有指定`el`，则需要手动使用`vm.$mount()`进行手动挂载实例
```js
new Vue({
	...
}).$mount('#app')

var vm = new Vue({
	...
}).$mount();
document.body.appendChild(vm.$el)
```
## 2. Vue.use
通过全局方法`Vue.use()`使用插件，`Vue.use()`会自动阻止多次注册相同插件，需要在你调用`new Vue()`启动应用之前完成
`Vue.use()`方法至少传入一个参数，该参数类型必须是`Object`或`Function`：如果是`Object`那么这个`Object`需要定义一个`install function`，如果是`Function`则会被当作`install function`，`Vue.use()`执行时会默认执行`install function`
该`install function`第一个参数是`Vue`，其他参数是`Vue.use()`执行时传入的其他参数

## 3. Vue.prototype
将变量、方法添加入全局作用域中，使所用`Vue`实例都能访问到
```js
Vue.prototype.$appName = 'My App'

new Vue({
	beforeCreate: function() {
		console.log(this.$appName)
	}
})
```

```js
// main.js
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'

Vue.config.productionTip = false
Vue.prototype.$appName = 'main'

new Vue({
    el: '#app',
    store,
    router,
    components: { App },
    template: '<App/>',
})
// 给所有组件注册了一个属性 $appName，赋予初始值 'main' ，所有组件都可以用 this.$appName 访问此变量;
// 如果组件中没有赋值，初始值都是'main'
```

```html
<!-- home.vue -->
<template>
  <div>
    <div @click="changeName">change name</div>
    <div @click="gotoTest2">goto test2</div>
  </div>
</template>

<script>
export default {
  methods:{
    changeName(){
      this.$appName = "test1"
    },
    gotoTest2(){
      this.$router.push('/about')
    } 
  }
}
</script>
```

```html
<!-- about.vue -->
<template>
  <div>
    <div>{{this.$appName}} in test2</div>
  </div>
</template>
```
## 4. Vue.component
`Vue.use`用于注册插件，`Vue.component`用于注册组件
## 5. Vue.directive
注册指令，即自定义指令
全局注册
```js
Vue.directive('dirName', function() {
	//定义指令
})
```

局部注册
```js
new Vue({
	directives: {
		dirName: {
			//定义指令
		}
	}
})
```

```js
Vue.directive('gqs',{
    bind() {
      // 当指令绑定到 HTML 元素上时触发.**只调用一次**
      console.log('bind triggerd')
    },
    inserted() {
      // 当绑定了指令的这个HTML元素插入到父元素上时触发(在这里父元素是 `div#app`)**.但不保证,父元素已经插入了 DOM 文档.**
      console.log('inserted triggerd')
    },
    updated() {
      // 所在组件的`VNode`更新时调用.
      console.log('updated triggerd')
    },
    componentUpdated() {
      // 指令所在组件的 VNode 及其子 VNode 全部更新后调用。
      console.log('componentUpdated triggerd')
      
    },
    unbind() {
      // 只调用一次，指令与元素解绑时调用.
      console.log('unbind triggerd')
    }
})
```
指令的声明周期以及钩子函数：
- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置
- `inserted`：被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）
- `update`：所在组件的`VNode`更新时调用，但是可能发生在其子`VNode`更新之前。指令的值可能发生了改变，也可能没有
- `componentUpdated`：指令所在组件的`VNode`及其子`VNode`全部更新后调用
- `unbind`：只调用一次，指令与元素解绑时调用

## 6. Vue.mixin
将组件的公共逻辑或者配置提取出来，哪个组件需要用到时，直接将提取的这部分混入到组件内部即可
与`Vuex`、`Pinia`区别：如果在一个组件中更改了`Vuex`中的某个数据，那么其它所有引用了`Vuex`中该数据的组件也会跟着变化；`Mixin`中的数据和方法都是独立的，组件之间使用后是互相不影响的
```js
// src/mixin/index.js
export const mixins = {
  data() {
    return {};
  },
  computed: {},
  created() {},
  mounted() {},
  methods: {},
};
```

全局混入
```js
import Vue from "vue";
import App from "./App.vue";
import { mixins } from "./mixin/index";
Vue.mixin(mixins);

Vue.config.productionTip = false;

new Vue({
  render: (h) => h(App),
}).$mount("#app");
```

局部混入
```html
// src/App.vue
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <button @click="clickMe">点击我</button>
  </div>
</template>

<script>
import { mixins } from "./mixin/index";
export default {
  name: "App",
  mixins: [mixins],
  components: {},
  created(){
    console.log("组件调用minxi数据",this.msg);
  },
  mounted(){
    console.log("我是组件的mounted生命周期函数")
  }
};
</script>
```

先执行`mixin`，后执行组件内部的生命周期函数