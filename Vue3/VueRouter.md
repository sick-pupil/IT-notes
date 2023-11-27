## 1. 安装使用
安装：`npm install vue-router@4`

使用：
```html
<script src="https://unpkg.com/vue@3"></script>
<script src="https://unpkg.com/vue-router@4"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!--使用 router-link 组件进行导航-->
    <!--通过传递 to 来指定链接-->
    <!--<router-link> 将呈现一个带有正确 href 属性的 <a> 标签-->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

- `router-link`：使用一个自定义组件`router-link`来创建链接。这使得`Vue Router`可以在不重新加载页面的情况下更改`URL`，处理`URL`的生成以及编码
- `router-view`：将显示与`url`对应的组件。你可以把它放在任何地方，以适应你的布局

```js
// 1. 定义路由组件.
// 也可以从其他文件导入
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. 定义一些路由
// 每个路由都需要映射到一个组件。
// 我们后面再讨论嵌套路由。
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. 创建路由实例并传递 routes 配置
// 你可以在这里输入更多的配置，但我们在这里
// 暂时保持简单
const router = VueRouter.createRouter({
  // 4. 内部提供了 history 模式的实现。为了简单起见，我们在这里使用 hash 模式。
  history: VueRouter.createWebHashHistory(),
  routes, // routes: routes 的缩写
})

// 5. 创建并挂载根实例
const app = Vue.createApp({})
//确保 _use_ 路由实例使
//整个应用支持路由。
app.use(router)

app.mount('#app')

// 现在，应用已经启动了！
```

通过调用`app.use(router)`，会触发第一次导航且可以在任意组件中以`this.$router`的形式访问它，并且以`this.$router`的形式访问当前路由
```js
// Home.vue
export default {
  computed: {
    username() {
      // 我们很快就会看到 params 是什么
      return this.$route.params.username
    },
  },
  methods: {
    goToDashboard() {
      if (isAuthenticated) {
        this.$router.push('/dashboard')
      } else {
        this.$router.push('/login')
      }
    },
  },
}
```
## 2. 带参动态路由
设置包含**匹配模式**的路由映射到同一个组件
```js
const User = {
  template: '<div>User</div>',
}

// 这些都会传递给 createRouter
const routes = [
  // 动态字段以冒号开始
  { path: '/users/:id', component: User },
]
```
**路径参数**用冒号`:`表示。当一个路由被匹配，则它的`params`值可以在每个组件中使用`this.$route.params`访问
```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```

可以设置**多个路径参数**，这些参数会映射到`$route.params`上的同名字段：

| 匹配模式 | 匹配路径 | $route.params |
| ----- | ----- | ----- |
| /users/:username | /users/eduardo | { username: 'eduardo' } |
| /users/:username/posts/:postId | /users/eduardo/posts/123 | { username: 'eduardo', postId: '123' } |

*除了\$route.params，还有\$route.query，\$router.hash*

```js
const User = {
  template: '...',
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        // 对路由变化做出响应...
      }
    )
  },
}
```

**可以使用高级匹配模式（使用自定义正则表达式）实现捕获特定路由**
## 3. 路由匹配语法
```js
const routes = [
  // /:orderId -> 仅匹配数字
  { path: '/:orderId(\\d+)' },
  // /:productName -> 匹配其他任何内容
  { path: '/:productName' },
]
```

```js
const routes = [
  // /:chapters ->  匹配 /one, /one/two, /one/two/three, 等
  { path: '/:chapters+' },
  // /:chapters -> 匹配 /, /one, /one/two, /one/two/three, 等
  { path: '/:chapters*' },
]
```

```js
// 给定 { path: '/:chapters*', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
// 产生 /
router.resolve({ name: 'chapters', params: { chapters: ['a', 'b'] } }).href
// 产生 /a/b

// 给定 { path: '/:chapters+', name: 'chapters' },
router.resolve({ name: 'chapters', params: { chapters: [] } }).href
// 抛出错误，因为 `chapters` 为空

const routes = [
  // 仅匹配数字
  // 匹配 /1, /1/2, 等
  { path: '/:chapters(\\d+)+' },
  // 匹配 /, /1, /1/2, 等
  { path: '/:chapters(\\d+)*' },
]
```
## 4. 嵌套路由
顶层组件：
```js
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `,
}
```
`/user:id`路由下还有`children`路由：
```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // 当 /user/:id/profile 匹配成功
        // UserProfile 将被渲染到 User 的 <router-view> 内部
        path: 'profile',
        component: UserProfile,
      },
      {
        // 当 /user/:id/posts 匹配成功
        // UserPosts 将被渲染到 User 的 <router-view> 内部
        path: 'posts',
        component: UserPosts,
      },
      {
	    path: '',
	    component: UserHome
	  },
    ],
  },
]
```

**针对路由声明路由名称**：
1. 指定页面路由，并传递参数；若使用`<router-link to="/xxx">`，则无法传递`params`参数
```html
<!-- 路由代码配置 -->
{
  path:'/Liantong/:id',
  component:Liantong,
  name:'LiantongName'
}

<!-- 页面导航跳转 -->
<router-link :to="{name:'LiantongName',params:{id:100}}">
  <el-menu-item index="/Liantong">
    <i class="el-icon-menu"></i>
    <span slot="title">联通数据</span>
  </el-menu-item>
</router-link>
```
2. 获取组件`name`提供被路由页面使用
```html
<!-- 路由代码配置 -->
{
  path:'/Yidong',
  component:Yidong,
  name:'我是移动name'
}

<!-- 页面渲染 -->
<template>
  <div class='container'>
    <h3>{{$route.name}}</h3>
  </div>
</template>
```
3. 同个路由渲染多个视图
```html
<!-- 路由代码配置 -->
{
  path:'/Dianxin',
  components:{
    default: DianxinOne, //default 默认的router-view名字
    DianxinTwo: DianxinTwo,
    DianxinThr: DianxinThr
  },
  name:'Dianxin'     
},
<div>
   <el-main>
      <router-view></router-view> //渲染默认DianxinOne组件
      <router-view name="DianxinTwo"></router-view> //渲染DianxinTwo组件
      <router-view name="DianxinThr"></router-view> //渲染DianxinThr组件
   </el-main>
</div>
```
## 5. 编程式导航
除了使用`<router-link>`创建`a`标签定义导航链接，还可以借助`router`的实例方法实现路由定义

可以通过`$router`访问路由实例，也可以调用`this.$router.push`导航到不同的`url`，向`history`栈添加一个新记录。因此，点击`<router-link>`时，相当于内部调用`router.push(...)`

| 声明式 | 编程式 |
| ----- | ----- |
| \<router-link :to="..."\> | router.push(...) |

```js
// 字符串路径
router.push('/users/eduardo')

// 带有路径的对象
router.push({ path: '/users/eduardo' })

// 命名的路由，并加上参数，让路由建立 url
router.push({ name: 'user', params: { username: 'eduardo' } })

// 带查询参数，结果是 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// 带 hash，结果是 /about#team
router.push({ path: '/about', hash: '#team' })
```

`router.push`和所有其他导航方法都会返回一个`Promise`，该`Promise`中包含路由结果

`router.replace`：替换当前位置，不同于`router.push`，它不会向路由`history`中添加新纪录
（也可以调用`router.push`时添加一个属性`replace: true`，`router.push({ path: '/home', replace: true })`）

| 声明式 | 编程式 |
| --- | --- |
| \<router-link :to="..." replace\> | router.replace(...) |

`window.history.go(n)`：横跨历史，在历史堆栈中前进或者后退n步
```js
// 向前移动一条记录，与 router.forward() 相同
router.go(1)

// 返回一条记录，与 router.back() 相同
router.go(-1)

// 前进 3 条记录
router.go(3)

// 如果没有那么多记录，静默失败
router.go(-100)
router.go(100)
```
## 6. 命名路由
除了`path`之外，还可以为路由提供`name`属性
```js
const routes = [
  {
    path: '/user/:username',
    name: 'user',
    component: User,
  },
]
```

声明式链接命名的路由
```html
<router-link :to="{ name: 'user', params: { username: 'erina' }}">User</router-link>
```

编程式链接命名的路由
```js
router.push({ name: 'user', params: { username: 'erina' } })
```

## 7. 命名视图
如果希望同级展示多个`router-view`，需要在`router-view`上设置名字`name`属性
```html
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>

<router-view class="view main-content"></router-view>

<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

```js
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar,
      },
    },
  ],
})
```

嵌套命名视图
```js
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

## 8. 重定向与别名
```js
const routes = [{ path: '/home', redirect: '/' }]

const routes = [{ path: '/home', redirect: { name: 'homepage' } }]

const routes = [
  {
    // /search/screens -> /search?q=screens
    path: '/search/:searchText',
    redirect: to => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
      return { path: '/search', query: { q: to.params.searchText } }
    },
  },
  {
    path: '/search',
    // ...
  },
]
```

重定向：用户访问`/home`，`url`被`/`替换，然后匹配`/`路由
别名：将`/`别名为`/home`，当用户访问`/home`，`url`仍然为`/home`，但被路由匹配`/`
```js
const routes = [{ path: '/', component: Homepage, alias: '/home' }]
```

```js
const routes = [
  {
    path: '/users',
    component: UsersLayout,
    children: [
      // 为这 3 个 URL 呈现 UserList
      // - /users
      // - /users/list
      // - /people
      { path: '', component: UserList, alias: ['/people', 'list'] },
    ],
  },
]

const routes = [
  {
    path: '/users/:id',
    component: UsersByIdLayout,
    children: [
      // 为这 3 个 URL 呈现 UserDetails
      // - /users/24
      // - /users/24/profile
      // - /24
      { path: 'profile', component: UserDetails, alias: ['/:id', ''] },
    ],
  },
]
```

## 9. prop传入路由参数
```js
const User = {
  // 请确保添加一个与路由参数完全相同的 prop 名
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

对于有命名视图的路由，你必须为每个命名视图定义`props`配置
```js
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
  }
]
```

当`props`是一个对象时，它将原样设置为组件 props。当 props 是静态的时候很有用
```js
const routes = [
  {
    path: '/promotion/from-newsletter',
    component: Promotion,
    props: { newsletterPopup: false }
  }
]
```

你可以创建一个返回`props`的函数。这允许你将参数转换为其他类型，将静态值与基于路由的值相结合等等
```js
const routes = [
  {
    path: '/search',
    component: SearchUser,
    props: route => ({ query: route.query.q })
  }
]
```
## 10. 不同的历史模式
- `Hash`模式
- `HTML5`模式
## 11. 导航守卫
```js
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // 返回 false 以取消导航
  return false
})
```

返回值：
- 取消当前的导航，如果浏览器的`URL`改变了，则会重置到`from`路由对应的地址
- 一个路由地址，通过一个路由地址跳转到一个不同的地址
```js
 router.beforeEach(async (to, from) => {
   if (
     // 检查用户是否已登录
     !isAuthenticated &&
     // ❗️ 避免无限重定向
     to.name !== 'Login'
   ) {
     // 将用户重定向到登录页面
     return { name: 'Login' }
   }
 })
```