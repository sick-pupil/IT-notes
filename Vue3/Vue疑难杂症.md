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

## 7. tsconfig.json
```json
// jsconfig.json
{
    "compilerOptions": {
        "target": "es2015",  // 指定要使用的默认库，值为"es3","es5","es2015"...
        "module": "commonjs", // 在生成模块代码时指定模块系统
        "checkJs": false, // 启用javascript文件的类型检查
        "baseUrl": "*", // 解析非相关模块名称的基础目录
        "paths": {
            "utils": ["src/utils/*"] // 指定相对于baseUrl选项计算的路径映射，使用webpack别名，智能感知路径
        }
    },
    "exclude": [ // 要排除的文件
        "node_modules", 
        "**/node_modules/*"
    ],
    "include": [ // 包含的文件
        "src/*.js"
    ]
}


"compilerOptions": {
  "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
  "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
  "diagnostics": true, // 打印诊断信息 
  "target": "ES5", // 目标语言的版本
  "module": "CommonJS", // 生成代码的模板标准
  "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件，可以用在AMD模块中，即开启时应设置"module": "AMD",
  "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
  "allowJS": true, // 允许编译器编译JS，JSX文件
  "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
  "outDir": "./dist", // 指定输出目录
  "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
  "declaration": true, // 生成声明文件，开启后会自动生成声明文件
  "declarationDir": "./file", // 指定生成声明文件存放目录
  "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
  "sourceMap": true, // 生成目标文件的sourceMap文件
  "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
  "declarationMap": true, // 为声明文件生成sourceMap
  "typeRoots": [], // 声明文件目录，默认时node_modules/@types
  "types": [], // 加载的声明文件包
  "removeComments":true, // 删除注释 
  "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
  "noEmitOnError": true, // 发送错误时不输出任何文件
  "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
  "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
  "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
  "strict": true, // 开启所有严格的类型检查
  "alwaysStrict": true, // 在代码中注入'use strict'
  "noImplicitAny": true, // 不允许隐式的any类型
  "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
  "strictFunctionTypes": true, // 不允许函数参数双向协变
  "strictPropertyInitialization": true, // 类的实例属性必须初始化
  "strictBindCallApply": true, // 严格的bind/call/apply检查
  "noImplicitThis": true, // 不允许this有隐式的any类型
  "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
  "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
  "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
  "noImplicitReturns": true, //每个分支都会有返回值
  "esModuleInterop": true, // 允许export=导出，由import from 导入
  "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
  "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
  "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
  "paths": { // 路径映射，相对于baseUrl
    // 如使用jq时不想使用默认版本，而需要手动指定版本，可进行如下配置
    "jquery": ["node_modules/jquery/dist/jquery.min.js"]
  },
  "rootDirs": ["src","out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
  "listEmittedFiles": true, // 打印输出文件
  "listFiles": true// 打印编译的文件(包括引用的声明文件)
}
```