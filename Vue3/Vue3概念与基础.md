## 1. 创建项目
```shell
npm create vue@latest

✔ Project name: … <your-project-name> 
✔ Add TypeScript? … No / Yes 
✔ Add JSX Support? … No / Yes 
✔ Add Vue Router for Single Page Application development? … No / Yes 
✔ Add Pinia for state management? … No / Yes 
✔ Add Vitest for Unit testing? … No / Yes 
✔ Add an End-to-End Testing Solution? … No / Cypress / Playwright 
✔ Add ESLint for code quality? … No / Yes 
✔ Add Prettier for code formatting? … No / Yes 

Scaffolding project in ./<your-project-name>... 
Done.

cd <your-project-name>
npm install
npm run dev
npm run build
```

使用`CDN`引入`Vue`
```js
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp } = Vue
  
  createApp({
    data() {
      return {
        message: 'Hello Vue!'
      }
    }
  }).mount('#app')
</script>

//-------------------使用ES模块-------------------
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script type="module">
  import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'
  
  createApp({
    data() {
      return {
        message: 'Hello Vue!'
      }
    }
  }).mount('#app')
</script>
```

使用ES拆分模块
```html
<!-- index.html -->
<div id="app"></div>

<script type="module">
  import { createApp } from 'vue'
  import MyComponent from './my-component.js'

  createApp(MyComponent).mount('#app')
</script>
```

```js
// my-component.js
export default {
  data() {
    return { count: 0 }
  },
  template: `<div>count is {{ count }}</div>`
}
```

每个`Vue`应用都是通过`createApp`函数创建一个新应用实例
```js
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

应用实例必须调用`.mount()`方法进行挂载，即将`Vue`根组件挂载并渲染在实际的`DOM`元素容器中，返回值为根组件实例而非应用实例

常见应用文件目录结构
```
App (root component)
├─ TodoList
│  └─ TodoItem
│     ├─ TodoDeleteButton
│     └─ TodoEditButton
└─ TodoFooter
   ├─ TodoClearButton
   └─ TodoStatistics
```

*可以配置一些应用级选项，如定义一个应用级的错误处理器，来捕获所有子组件上的错误*
```js
app.config.errorHandler = (err) => {
  /* 处理错误 */
}
```

*注册应用范围内可用的资源，如注册一个组件*
```js
app.component('TodoDeleteButton', TodoDeleteButton)
```

## 2. 模板语法

### 1. 文本插值
数据来源于`createApp`中的`data()`
```html
<span>Message: {{ msg }}</span>
```

### 2. 原始HTML
```html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

### 3. Attribute绑定
```html
<div v-bind:id="dynamicId"></div>
<!-- v-bind缩写 -->
<div :id="dynamicId"></div>

<!-- 布尔值Attr，可以决定该Attr是否存在于该元素上 -->
<button :disabled="isButtonDisabled">Button</button>
```

*动态绑定多个值*
```js
data() {
  return {
    objectOfAttrs: {
      id: 'container',
      class: 'wrapper'
    }
  }
}
```

```html
<div v-bind="objectOfAttrs"></div>
```

### 4. Mustache的JS表达式
```js
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>

//这是一个语句，而非表达式
{{ var a = 1 }}

//条件控制也不支持，请使用三元表达式
{{ if (ok) { return message } }}

//调用函数
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

### 5. 指令Directives
*v-if*
```html
<p v-if="seen">Now you see me</p>
```

*v-on*
```html
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

*v-bind与v-on动态绑定属性*
```html
<!-- attributeName为组件内的data()响应式属性 -->
<a v-bind:[attributeName]="url"> ... </a>
<!-- 简写 -->
<a :[attributeName]="url"> ... </a>

<!-- eventName为组件内的data()响应式属性 -->
<a v-on:[eventName]="doSomething"> ... </a>
<!-- 简写 -->
<a @[eventName]="doSomething">
```

## 3. 声明方法
```js
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  mounted() {
    // 在其他方法或是生命周期中也可以调用方法
    this.increment()
  }
}
```

```html
<button @click="increment">{{ count }}</button>
```

## 4. 计算属性

组件中的响应式数据只要发生改变，组件状态以及`DOM`也会自动更新；但`DOM`更新是不同步的，`Vue`会在`next tick`更新周期中缓冲所有状态的修改，在这个周期不管你修改过多少次状态，每个组件都只会被更新一次
```js
import { nextTick } from 'vue'

export default {
  methods: {
    async increment() {
      this.count++
      await nextTick()
      // 现在 DOM 已经更新了
    }
  }
}
```

而`computed`计算属性也如此，当响应式数据不发生变化则每次调用都是取缓存值，当响应式数据发生变化则会进行计算并将新结果缓存
```js
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    // 一个计算属性的 getter
    publishedBooksMessage() {
      // `this` 指向当前组件实例
      return this.author.books.length > 0 ? 'Yes' : 'No'
    }
  }
}
```

## 5. Class/Style绑定
### 1. 响应式数据与Class绑定
```js
data() {
  return {
    isActive: true,
    hasError: false
  }
}

data() {
  return {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
}

data() {
  return {
    isActive: true,
    error: null
  }
},
computed: {
  classObject() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}

data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
}
```

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>

<div :class="classObject"></div>

<div :class="[activeClass, errorClass]"></div>
```

渲染结果
```html
<div class="static active"></div>

<div class="active"></div>

<div class="active text-danger"></div>
```

### 2. 响应式数据与style绑定
```js
data() {
  return {
    activeColor: 'red',
    fontSize: 30
  }
}

data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}
```

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

<div :style="styleObject"></div>
```

## 6. 条件渲染

### 1. 切换单个元素
```html
<h1 v-if="awesome">Vue is awesome!</h1>


<button @click="awesome = !awesome">Toggle</button>
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>


<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

### 2. 切换多个元素
```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

### 3. 隐藏元素
```html
<h1 v-show="ok">Hello!</h1>
```

`v-show`会在`DOM`渲染中保留该元素；`v-show`仅切换了该元素上名为`display`的`CSS`属性；`v-show`不支持在`<template>`元素上使用，也不能和`v-else`搭配使用

`v-if`与`v-show`区别：
1. `v-if` 是“真实的”按条件渲染，因为它确保了在切换时，条件区块内的事件监听器和子组件都会被销毁与重建
2. `v-if` 也是**惰性**的：如果在初次渲染时条件值为 false，则不会做任何事。条件区块只有当条件首次变为 true 时才被渲染
3. 相比之下，`v-show` 简单许多，元素无论初始条件如何，始终会被渲染，只有 CSS `display` 属性会被切换

*当`v-if`和`v-for`同时存在于一个元素上的时候，`v-if`会首先被执行*

## 7. 列表渲染
### 1. v-for数组渲染
```js
data() {
  return {
    parentMessage: 'Parent',
    items: [{ message: 'Foo' }, { message: 'Bar' }]
  }
}
```

```html
<li v-for="(item, index) in items">
  {{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

```
Parent - 0 - Foo
Parent - 1 - Bar
```


```html
<li v-for="{ message } in items">
  {{ message }}
</li>

<!-- 有 index 索引时 -->
<li v-for="({ message }, index) in items">
  {{ message }} {{ index }}
</li>
```


```html
<li v-for="item in items">
  <span v-for="childItem in item.children">
    {{ item.message }} {{ childItem }}
  </span>
</li>
```

### 2. v-for对象渲染
```js
data() {
  return {
    myObject: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
}
```

```html
<ul>
  <li v-for="value in myObject">
    {{ value }}
  </li>
</ul>

<li v-for="(value, key) in myObject">
  {{ key }}: {{ value }}
</li>

<li v-for="(value, key, index) in myObject">
  {{ index }}. {{ key }}: {{ value }}
</li>
```

### 3. v-for范围值
```html
<span v-for="n in 10">{{ n }}</span>
```

### 4. v-for与template
```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

### 5. v-for与key
`Vue`默认按照“就地更新”的策略来更新通过`v-for`渲染的元素列表。当数据项的顺序改变时，`Vue`不会随之移动`DOM`元素的顺序，而是就地更新每个元素，确保它们在原本指定的索引位置上渲染

为了给`Vue`一个提示，以便它可以跟踪每个节点的标识，从而重用和重新排序现有的元素，你需要为每个元素对应的块提供一个唯一的 `key``attribute`
```html
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```

```html
<template v-for="todo in todos" :key="todo.name">
  <li>{{ todo.name }}</li>
</template>
```

*组件使用v-for*
```html
<MyComponent
  v-for="(item, index) in items"
  :item="item"
  :index="index"
  :key="item.id"
/>
```

*`Vue`能够侦听响应式数组的变更方法，并在它们被调用时触发相关更新*
- `push`
- `pop`
- `shift`
- `unshift`
- `splice`
- `sort`
- `reverse`

## 8. 事件处理
- 内联事件处理：`<button @click="count++">Add 1</button>`
- 方法事件处理：
```html
<!-- `greet` 是上面定义过的方法名 -->
<button @click="greet">Greet</button>
```

```js
data() {
  return {
    name: 'Vue.js'
  }
},
methods: {
  greet(event) {
    // 方法中的 `this` 指向当前活跃的组件实例
    alert(`Hello ${this.name}!`)
    // `event` 是 DOM 原生事件
    if (event) {
	//能够通过被触发事件的event.target.tagName访问到该DOM元素
      alert(event.target.tagName)
    }
  }
}
```

在内联事件处理中：
1. 调用自定义方法：`<button @click="say('hello')">Say hello</button>`
2. 访问事件参数：
```html
<!-- 使用特殊的 $event 变量 -->
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>

<!-- 使用内联箭头函数 -->
<button @click="(event) => warn('Form cannot be submitted yet.', event)">
  Submit
</button>
```

```js
methods: {
  warn(message, event) {
    // 这里可以访问 DOM 原生事件
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

使用事件修饰符
```html
<!-- 单击事件将停止传递 -->
<a @click.stop="doThis"></a>

<!-- 提交事件将不再重新加载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰语可以使用链式书写 -->
<a @click.stop.prevent="doThat"></a>

<!-- 也可以只有修饰符 -->
<form @submit.prevent></form>

<!-- 仅当 event.target 是元素本身时才会触发事件处理器 -->
<!-- 例如：事件处理器不来自子元素 -->
<div @click.self="doThat">...</div>

<!-- 添加事件监听器时，使用 `capture` 捕获模式 -->
<!-- 例如：指向内部元素的事件，在被内部元素处理前，先被外部处理 -->
<div @click.capture="doThis">...</div>

<!-- 点击事件最多被触发一次 -->
<a @click.once="doThis"></a>

<!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
<!-- 以防其中包含 `event.preventDefault()` -->
<div @scroll.passive="onScroll">...</div>
```

使用按键修饰符
```html
<!-- 仅在 `key` 为 `Enter` 时调用 `submit` -->
<input @keyup.enter="submit" />

<input @keyup.page-down="onPageDown" />

<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + 点击 -->
<div @click.ctrl="doSomething">Do something</div>
```
- `.enter`
- `.tab`
- `.delete`
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`
- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

## 9. 表单
常常需要将表单输入框的内容同步给`JavaScript`中相应的变量
```html
<input
  :value="text"
  @input="event => text = event.target.value">
```
等价于
```html
<input v-model="text">
```

`v-model`会根据所使用的元素自动使用对应的`DOM`属性和事件组合
- 文本类型的`<input>`和`<textarea>`元素会绑定`value` `property`并侦听`input`事件
- `<input type="checkbox">`和`<input type="radio">`会绑定`checked` `property`并侦听 `change` 事件
- `<select>`会绑定`value` `property`并侦听`change`事件

### 1. 文本
```html
<p>Message is: {{ message }}</p>
<input v-model="message" placeholder="edit me" />
```

### 2. 多行文本
```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

### 3. 复选框
```html
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label>
```

```js
export default {
  data() {
    return {
      checkedNames: []
    }
  }
}
```

```html
<div>Checked names: {{ checkedNames }}</div>

<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>

<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>

<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
```

`value`与响应式数据绑定
```html
<input
  type="checkbox"
  v-model="toggle"
  :true-value="dynamicTrueValue"
  :false-value="dynamicFalseValue" />
```

### 4. 单选框
```html
<div>Picked: {{ picked }}</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```

```html
<input type="radio" v-model="pick" :value="first" />
<input type="radio" v-model="pick" :value="second" />
```

### 5. 选择器
```html
<div>Selected: {{ selected }}</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

```js
export default {
  data() {
    return {
      selected: 'A',
      options: [
        { text: 'One', value: 'A' },
        { text: 'Two', value: 'B' },
        { text: 'Three', value: 'C' }
      ]
    }
  }
}
```

```html
<select v-model="selected">
  <option v-for="option in options" :value="option.value">
    {{ option.text }}
  </option>
</select>

<div>Selected: {{ selected }}</div>
```

### 6. 修饰符
- `.lazy`：`v-model`会在每次`input`事件后更新数据
```html
<!-- 在 "change" 事件后同步更新而不是 "input" -->
<input v-model.lazy="msg" />
```
- `.number`：让用户输入自动转换为数字，你可以在`v-model`后添加`.number`修饰符来管理输入
```html
<input v-model.number="age" />
```
- `.trim`：默认自动去除用户输入内容中两端的空格
```html
<input v-model.trim="msg" />
```

## 10. 生命周期
<img src="D:\Project\IT-notes\Vue3\img\Vue3生命周期.png" style="width:700px;height:1100px;" />

## 11. 侦听器
```js
export default {
  data() {
    return {
      question: '',
      answer: 'Questions usually contain a question mark. ;-)'
    }
  },
  watch: {
    // 每当 question 改变时，这个函数就会执行
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes('?')) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.answer = 'Thinking...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Error! Could not reach the API. ' + error
      }
    }
  }
}
```

`watch`默认是浅层的：被侦听的属性，仅在被赋新值时，才会触发回调函数——而嵌套属性的变化不会触发。如果想侦听所有嵌套的变更，需要深层侦听器
```js
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // 注意：在嵌套的变更中，
        // 只要没有替换对象本身，
        // 那么这里的 `newValue` 和 `oldValue` 相同
      },
      deep: true
    }
  }
}
```

`watch`默认是懒执行的：仅当数据源变化时，才会执行回调。但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调
```js
export default {
  // ...
  watch: {
    question: {
      handler(newQuestion) {
        // 在组件实例创建时会立即调用
      },
      // 强制立即执行回调
      immediate: true
    }
  }
  // ...
}
```
*回调函数的初次执行就发生在`created`钩子之前。`Vue`此时已经处理了`data`、`computed`和`methods`选项，所以这些属性在第一次调用时就是可用的*

**默认情况下，用户创建的侦听器回调，都会在 Vue 组件更新**之前**被调用。这意味着你在侦听器回调中访问的 DOM 将是被 Vue 更新之前的状态**
**如果想在侦听器回调中能访问被 Vue 更新**之后**的 DOM，你需要指明 `flush: 'post'` 选项**
```js
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'post'
    }
  }
}
```

也可以使用组件实例的`$watch()`来命令式地创建一个侦听器
```js
export default {
  created() {
    this.$watch('question', (newQuestion) => {
      // ...
    })
  }
}
```

## 12. 模板引用
暴露子组件`this`的引用
```js
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child 是 <Child /> 组件的实例
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

## 13. 组件
子组件：
```html
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>
```

```js
export default {
  data() {
    return {
      count: 0
    }
  },
  template: `
    <button @click="count++">
      You clicked me {{ count }} times.
    </button>`
}
```


父组件：
```js
<script>
import ButtonCounter from './ButtonCounter.vue'

export default {
  components: {
    ButtonCounter
  }
}
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

若要将导入的组件暴露给模板，我们需要在`components`选项上注册它。这个组件将会以其注册时的名字作为模板中的标签名
当然，也可以全局地注册一个组件，使得它在当前应用中的任何组件上都可以使用，而不需要额外再导入

### 1. 传递数据进入子组件
子组件定义：
```js
<!-- BlogPost.vue -->
<script>
export default {
  props: ['title']
}
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

```html
<BlogPost title="My journey with Vue" />
<BlogPost title="Blogging with Vue" />
<BlogPost title="Why Vue is so fun" />
```

父组件使用：
```js
export default {
  // ...
  data() {
    return {
      posts: [
        { id: 1, title: 'My journey with Vue' },
        { id: 2, title: 'Blogging with Vue' },
        { id: 3, title: 'Why Vue is so fun' }
      ]
    }
  }
}
```

```html
<BlogPost
  v-for="post in posts"
  :key="post.id"
  :title="post.title"
 />
```

### 2. 子组件调用父组件方法
父组件：
```html
<div :style="{ fontSize: postFontSize + 'em' }">
  <BlogPost
    v-for="post in posts"
    :key="post.id"
    :title="post.title"
    @enlarge-text="postFontSize += 0.1"
   />
</div>
```

子组件：
```html
<!-- BlogPost.vue, 省略了 <script> -->
<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <button @click="$emit('enlarge-text')">Enlarge text</button>
  </div>
</template>
```

### 3. 插槽
希望子组件可以存在**插槽**给与父组件传入内容
子组件：
```html
<template>
  <div class="alert-box">
    <strong>This is an Error for Demo Purposes</strong>
    <slot />
  </div>
</template>

<style scoped>
.alert-box {
  /* ... */
}
</style>
```

父组件：
```html
<AlertBox>
  Something bad happened.
</AlertBox>
```

### 4. 动态组件
多个组件动态切换
```html
<!-- currentTab 改变时组件也改变 -->
<component :is="currentTab"></component>
```

`:is`值可以为以下几种：
- 被注册的组件名
- 导入的组件对象

