## 1. 注册
### 1. 全局注册
让组件在当前`Vue`应用中全局可用
```js
import { createApp } from 'vue'

const app = createApp({})
app.component(
  // 注册的名字
  'MyComponent',
  // 组件的实现
  {
    /* ... */
  }
)
```

```js
import MyComponent from './App.vue'

app.component('MyComponent', MyComponent)

app
  .component('ComponentA', ComponentA)
  .component('ComponentB', ComponentB)
  .component('ComponentC', ComponentC)
```

```html
<!-- 这在当前应用的任意组件中都可用 -->
<ComponentA/>
<ComponentB/>
<ComponentC/>
```
所有的子组件如果都使用全局注册，则所有子组件都处于同一作用域内，可以彼此互相使用
### 2. 局部注册
局部注册的组件需要在使用它的父组件中显式导入，并且只能在该父组件中使用。它的优点是使组件之间的依赖关系更加明确

- 在使用`<script setup>`的单文件组件中，导入的组件可以直接在模板中使用，无需注册
```js
<script setup>
import ComponentA from './ComponentA.vue'
</script>

<template>
  <ComponentA />
</template>
```
- 如果没有使用`<script setup>`，则需要使用`components`显式注册
```js
import ComponentA from './ComponentA.js'

export default {
  components: {
    ComponentA
  },
  setup() {
    // ...
  }
}
//等价于
export default {
  components: {
    ComponentA: ComponentA
  }
  // ...
}
```

组件名格式：一个以`MyComponent`命名注册的组件，在模板中可以通过`<MyComponent>`与`<my-component>`进行引用

## 2. Props
使用`props`从外部接收传参进入组件

- 在使用`<script setup>`的单文件组件中，`props`可以使用`defineProps()`宏声明
```html
<script setup>
const props = defineProps(['foo'])

/**
// 使用 <script setup>
defineProps({
  title: String,
  likes: Number
})
**/

console.log(props.foo)
</script>
```
- 在没有使用`<script setup>`的组件中，使用`props`选项声明`prop`
```js
export default {
  /**
  props: { title: String, likes: Number }
  **/
  props: ['foo'],
  setup(props) {
    // setup() 接收 props 作为第一个参数
    console.log(props.foo)
  }
}
```

`props`传值例子
```html
<!-- 虽然 `42` 是个常量，我们还是需要使用 v-bind -->
<!-- 因为这是一个 JavaScript 表达式而不是一个字符串 -->
<BlogPost :likes="42" />

<!-- 根据一个变量的值动态传入 -->
<BlogPost :likes="post.likes" />

<!-- 仅写上 prop 但不传值，会隐式转换为 `true` -->
<BlogPost is-published />

<!-- 虽然 `false` 是静态的值，我们还是需要使用 v-bind -->
<!-- 因为这是一个 JavaScript 表达式而不是一个字符串 -->
<BlogPost :is-published="false" />

<!-- 根据一个变量的值动态传入 -->
<BlogPost :is-published="post.isPublished" />

<!-- 虽然这个数组是个常量，我们还是需要使用 v-bind -->
<!-- 因为这是一个 JavaScript 表达式而不是一个字符串 -->
<BlogPost :comment-ids="[234, 266, 273]" />

<!-- 根据一个变量的值动态传入 -->
<BlogPost :comment-ids="post.commentIds" />

<!-- 虽然这个对象字面量是个常量，我们还是需要使用 v-bind -->
<!-- 因为这是一个 JavaScript 表达式而不是一个字符串 -->
<BlogPost
  :author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
 />

<!-- 根据一个变量的值动态传入 -->
<BlogPost :author="post.author" />
```

对传入的`props`进行类型校验
```js
defineProps({
  // 基础类型检查
  // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
  propA: Number,
  // 多种可能的类型
  propB: [String, Number],
  // 必传，且为 String 类型
  propC: {
    type: String,
    required: true
  },
  // Number 类型的默认值
  propD: {
    type: Number,
    default: 100
  },
  // 对象类型的默认值
  propE: {
    type: Object,
    // 对象或数组的默认值
    // 必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // 自定义类型校验函数
  propF: {
    validator(value) {
      // The value must match one of these strings
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // 函数类型的默认值
  propG: {
    type: Function,
    // 不像对象或数组的默认，这不是一个
    // 工厂函数。这会是一个用来作为默认值的函数
    default() {
      return 'Default function'
    }
  }
})
```

## 3. 触发&监听事件
```html
<!-- 子组件内容 -->
<button @click="$emit('increaseBy', 1)">
  Increase by 1
</button>
```

```html
<!-- 父组件调用子组件 -->
<MyButton @increase-by="(n) => count += n" />

<!-- 
	<MyButton @increase-by="increaseCount" /> 

	function increaseCount(n) {
	  count.value += n
	}
-->
```