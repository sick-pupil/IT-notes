## 1. 安装与使用
```shell
yarn add pinia
# 或者使用 npm
npm install pinia
```

```js
// main.ts
import { createApp } from "vue"
import App from "./App.vue"
import { createPinia } from "pinia"

const pinia = createPinia()
const app = createApp(App)
app.use(pinia)
app.mount("#app")
```

```js
import { defineStore } from 'pinia'

// 第一个参数是应用程序中 store 的唯一 id
export const useUsersStore = defineStore('users', {
  // 其它配置项
})
```

创建`store`时接收两个参数：
- `name`：一个字符串，必传项，该`store`的唯一`id`
- `options`：一个对象，`store`配置项

```html
<script setup lang="ts">
	import { useUsersStore } from "../src/store/user";
	const store = useUsersStore();
	console.log(store);
</script>
```

<img src="D:\Project\IT-notes\Vue3\img\Pinia Store变量.png" style="width:700px;height:300px;" />

## 2. 存放数据
例：向`store`存储`state`，`state`中添加数据
```js
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪课堂",
      age: 25,
      sex: "男",
    };
  },
});
```
## 3. 操作数据
### 1. 读取数据
```html
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p>姓名：{{ name }}</p>
  <p>年龄：{{ age }}</p>
  <p>性别：{{ sex }}</p>
</template>
<script setup lang="ts">
	import { ref } from "vue";
	import { useUsersStore } from "../src/store/user";
	const store = useUsersStore();
	const name = ref<string>(store.name);
	const age = ref<number>(store.age);
	const sex = ref<string>(store.sex);
</script>
```
### 2. 修改数据
将`store`中的数据更改为响应式的，后修改数据
```html
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p>姓名：{{ name }}</p>
  <p>年龄：{{ age }}</p>
  <p>性别：{{ sex }}</p>
  <button @click="changeName">更改姓名</button>
</template>
<script setup lang="ts">
	import child from './child.vue';
	import { useUsersStore } from "../src/store/user";
	import { storeToRefs } from 'pinia';
	const store = useUsersStore();
	const { name, age, sex } = storeToRefs(store);
	const changeName = () => {
	  store.name = "张三";
	  console.log(store);
	};
</script>
```

重置`state`
```js
<button @click="reset">重置store</button>
// 重置store
const reset = () => {
  store.$reset();
};
```

批量更改`state`
```js
//将state中所有字段都修改
<button @click="patchStore">批量修改数据</button>
// 批量修改数据
const patchStore = () => {
  store.$patch({
    name: "张三",
    age: 100,
    sex: "女",
  });
};

//将state中部分字段都修改
store.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```

替换整个`state`
```js
store.$state = { counter: 666, name: '张三' }
```

`getters`属性
*可以把`getter`想象成`Vue`中的计算属性，它的作用就是返回一个新的结果，既然它和`Vue`中的计算属性类似，那么它肯定也是会被缓存的，就和`computed`一样*
```js
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪课堂",
      age: 25,
      sex: "男",
    };
  },
  getters: {
    getAddAge: (state) => {
      return state.age + 100;
    },
  },
});
```

```html
<template>
  <p>新年龄：{{ store.getAddAge }}</p>
  <button @click="patchStore">批量修改数据</button>
</template>
<script setup lang="ts">
import { useUsersStore } from "../src/store/user";
const store = useUsersStore();
// 批量修改数据
const patchStore = () => {
  store.$patch({
    name: "张三",
    age: 100,
    sex: "女",
  });
};
</script>
```

```js
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪课堂",
      age: 25,
      sex: "男",
    };
  },
  getters: {
    getAddAge: (state) => {
      return state.age + 100;
    },
    getNameAndAge(): string {
      return this.name + this.getAddAge; // 调用其它getter
    },
  },
});
```

`action`属性
*有业务代码的话，最好就是写在`actions`属性里面，该属性就和我们组件代码中的`methods`相似，用来放置一些处理业务逻辑的方法*
```js
export const useUsersStore = defineStore("users", {
  state: () => {
    return {
      name: "小猪课堂",
      age: 25,
      sex: "男",
    };
  },
  getters: {
    getAddAge: (state) => {
      return (num: number) => state.age + num;
    },
    getNameAndAge(): string {
      return this.name + this.getAddAge; // 调用其它getter
    },
  },
  actions: {
    saveName(name: string) {
      this.name = name;
    },
  },
});
```