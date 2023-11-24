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

```js
<!-- 父组件 -->
//getGatewayData要获取的参数
<tree :show="show" @gatewayData="getGatewayData"></tree>
//执行方法获取参数
const getGatewayData = (e) => {
  console.log('getGatewayData', e) // 这里的e，就是子组件传过来的值label.value
}
```

```js
<!-- 子组件 -->
import { ref, defineEmits } from 'vue'
const emits = defineEmits(['gatewayData'])
const handleNodeClick = (e) => {
 emits('gatewayData', label.value)
}
```
## 4. v-model
实现组件内容与数据的双向绑定

`v-model`在原生元素上的用法：`<input v-model="searchText" />`，该用法相当于：
```html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```

而在自定义组件上使用`v-model`会被展开为以下格式：
```html
<CustomInput
  :model-value="searchText"
  @update:model-value="newValue => searchText = newValue"
/>
```
**自定义组件实现双向绑定需要做两件事**：
1. 将内部原生`<input>`元素的`value`属性绑定到`modelValue`命名的`prop`
2. 当原生的`input`事件触发时，触发一个携带了新值的`update:modelValue`自定义事件
```html
<!-- CustomInput.vue -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

**如果想要实现v-model使用自定义的绑定属性，则可以通过以下方法实现**
```html
<MyComponent v-model:title="bookTitle" />

<!-- MyComponent.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

## 5. Attribute透传
当一个组件以单个元素为根作渲染时，透传的 attribute 会自动被添加到根元素上
```html
<MyButton class="large" />
<!-- <MyButton> 的模板 -->
<button>click me</button>
<!-- 最后渲染出的结果DOM -->
<button class="large">click me</button>
```

`class`与`style`会合并
```html
<!-- <MyButton> 的模板 -->
<button class="btn">click me</button>
<!-- 最后渲染出的DOM -->
<button class="btn large">click me</button>
```

`v-on`也一样会被继承，子组件的原生`button`被触发点击事件后，也会触发父组件的点击事件
```html
<MyButton @click="onClick" />
<button @click="onClick">click me</button>
```

禁用`attribute`透传，使用`inheritAttrs`
```html
<script setup>
defineOptions({
  inheritAttrs: false
})
// ...setup 逻辑
</script>
```

`js`访问透传`attr`
```html
<span>Fallthrough attribute: {{ $attrs }}</span>

<!-- 子组件模板 -->
<div class="btn-wrapper">
  <button class="btn">click me</button>
</div>
<!-- 透传attr后 -->
<div class="btn-wrapper">
  <button class="btn" v-bind="$attrs">click me</button>
</div>
```
## 6. 插槽
<img src="D:\Project\IT-notes\Vue3\img\插槽.png" style="width:700px;height:280px;" />
设置插槽默认内容
```html
<!-- 子组件内容 -->
<button type="submit">
  <slot>
    Submit <!-- 默认内容 -->
  </slot>
</button>

<SubmitButton />
<!-- 等价于 -->
<button type="submit">Submit</button>

<SubmitButton>Save</SubmitButton>
<!-- 等价于 -->
<button type="submit">Save</button>
```

具名插槽：当希望一个子组件配置多个插槽时，使用`name`命名插槽，没有提供`name`命名的则隐式命名为`default`
```html
<!-- 子组件模板 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

```html
<!-- 具名插槽使用 -->
<BaseLayout>
  <!-- 简写 <template #header> -->
  <template v-slot:header>
    <!-- header 插槽的内容放这里 -->
  </template>
</BaseLayout>
```

动态插槽名
```html
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>

  <!-- 缩写为 -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

子组件向父组件插槽传参（默认插槽）
```html
<!-- <MyComponent> 的模板 -->
<div>
  <slot :text="greetingMessage" :count="1"></slot>
</div>

<!-- 父组件调用并传参 -->
<MyComponent v-slot="slotProps">
  {{ slotProps.text }} {{ slotProps.count }}
</MyComponent>
```

子组件向父组件插槽传参（具名插槽）
```html
<!-- 子组件定义插槽 -->
<!-- 传递message: hello -->
<slot name="header" message="hello"></slot>

<!-- 父组件调用 -->
<MyComponent>
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

## 7. 依赖注入
在单组件应用系统中，存在一个组件树，而如果希望爷孙组件进行数据传递，使用`prop`与`emit`层层传递是十分不方便的，此时可以使用`provide`与`inject`实现数据依赖注入

```js
import { createApp } from 'vue'

const app = createApp({})

app.provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
```

```html
<script setup>
import { inject } from 'vue'

const message = inject('message')
</script>
```

```js
import { inject } from 'vue'

export default {
  setup() {
    const message = inject('message')
    return { message }
  }
}
```

## 8. 异步组件
```js
import { defineAsyncComponent } from 'vue'
const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    // ...从服务器获取组件
    resolve(/* 获取到的组件 */)
  })
})
// ... 像使用其他一般组件一样使用 `AsyncComp`



import { defineAsyncComponent } from 'vue'
const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)

app.component('MyComponent', defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
))

<script setup>
import { defineAsyncComponent } from 'vue'
const AdminPage = defineAsyncComponent(() =>
  import('./components/AdminPageComponent.vue')
)
</script>
<template>
  <AdminPage />
</template>
```

加载与错误状态
```js
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),

  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,
  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
```