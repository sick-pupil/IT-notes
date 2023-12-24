## 1. 构建工具
**正常构建过程**：
1. 将源码的`typescript`使用`tsc`编译转换为`javascript`代码
2. 安装并使用`react-compiler/vue-compiler`编译`.jsx/.vue`文件转换为`render`函数
3. 安装并使用`less-loader/sass-loader`编译`less/sass/postcss/component-style`语法为普通的`css`
4. 进行语法降级，使用`babel`将`ES`新语法向旧版浏览器兼容
5. 代码包体积优化，`uglifyjs`压缩代码

`webpack`支持多种模块化语法，包括`commonJS`与`ESModule`，也正因为`webpack`需要将多种模块化语法编译统一，`webpack`需要将所有依赖都读取一遍

而`vite`默认使用的`ESModule`语法

## 2. 安装使用
1. `npm create vite@latest my-vue-app -- --template vue`新建模板项目
2. 在模板项目根目录下执行`npx vite`

**依赖预构建：**
1. 解决不同第三方包的不同导出格式语法规范：`vite`找到对应依赖，调用esbuild对其他规范（`commonJS、es5、es3`）代码转换为`ESModule`规范，并把转换结果存放在`node_modules/.vite/deps`目录
2. 解决对路径上的处理直接使用`.vite/deps`：在处理过程中如果存在非绝对路径或者相对路径的引用，则`vite`会尝试路径补全，向上找寻依赖直至搜索到根目录或目标依赖为止
3. 网络多包传输：将所有依赖模块统一集成在几个`js`中，避免传输过多的依赖文件
## 3. 配置文件
```ts
//导出一个默认的配置对象，其中包括了 Vite 的各种配置选项。
export default defineConfig({
    root, //项目根目录（index.html 文件所在的位置） 默认： process.cwd()
    base: '/', //  开发或生产环境服务的公共基础路径：默认'/'   1、绝对 URL 路径名： /foo/；  2、完整的 URL： https://foo.com/； 3、空字符串或 ./（用于开发环境）
    publicDir: resolve(__dirname, './dist'), //默认'public'  作为静态资源服务的文件夹  (打包public文件夹会没有，里面得东西会直接编译在dist文件下)
    assetsInclude: resolve(__dirname, './src/assets'), // 静态资源处理
    
    /*****配置项目的构建过程******/
    build: {
        outDir: 'dist', // 构建输出目录
        minify: true, // 是否压缩代码
        sourcemap: true, // 是否生成 source map
    },
    
    /*****定义全局变量******/
    define: {
        MENU_PATH: `"path"`,
        MENU_SHOW: `"isShowOnMenu"`,
        MENU_KEEPALIVE: `"keepAlive"`,
        MENU_KEY: `"key"`,
        MENU_ICON: `"icon"`,
        MENU_TITLE: `"title"`,
        MENU_CHILDREN: `"children"`,
        MENU_PARENTKEY: `"parentKey"`,
        MENU_ALLPATH: `"allPath"`,
        MENU_PARENTPATH: `"parentPath"`,
        MENU_LAYOUT: `'layout'`,
        __IS_THEME__: `${process.env.REACT_APP_COLOR === "1"}`,
        CUSTOMVARLESSDATA: `${JSON.stringify(customVarLessJson)}`
    },
    
    /******配置插件******/
    plugins: [
        ReactRouterGenerator({
          outputFile: resolve(".", "./src/router/auto.jsx"),
          isLazy: true,
          comKey: "components"
        }),
       react(),
    ],
    
 
    /******配置模块解析的规则******/
    resolve: {
      //路径使用别名
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url)),
      },
      //引入文件的后缀名称，可以省略。如果出现同名，按照数组加载的优先顺序
      extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json', '.vue'],
    },
    
    /******配置开发服务器******/
    server: {
      port: 3333,// 端口号
      open: true,// 启动时自动在浏览器打开
      https: true,// 是否开启 https
      host: true, // 监听所有地址
      cors: false, //为开发服务器配置 CORS
      fs: {
        // 可以为项目根目录的上一级提供服务
        allow: [".."],
      },
      //配置自定义代理规则
      proxy: {
          '^/api': {
                target: "https://z3web.cn",
                changeOrigin: true,
                rewrite: (path) => {
                return path.replace("/api", "/api/react-ant-admin")
            }
       },
    },
    
    /*****配置CSS相关的选项********/
    css: {
      //配置了对 SCSS 的处理选项
      preprocessorOptions: {
        scss: {
          //引入了全局的 SCSS 文件 global.scss
          additionalData: `@import "./src/assets/css/global.scss";`,
        },
      },
      // 可以查看 CSS 的源码
      devSourcemap: true
    },
    
    /****配置 esbuild 相关的选项******/
    esbuild: {
      // 自定义 JSX 配置
      jsxFactory: 'h', //自定义的 JSX 工厂函数为 h，这在一些非 React 框架中可能会用到。
      jsxFragment: 'Fragment', //指定了 JSX 的 Fragment 为 Fragment
      jsxInject: `import React from 'react'` //是否开启 JSX 转换
    }
})
```
## 4. CSS Module
- `CSS`自动导入：导入`.css`文件则会把内容插入`<style>`标签中
- 任何以`.module.css`为后缀名的`css`文件都被认为是一个`css module`
```
/* example.module.css */
.red {
  color: red;
}

import classes from './example.module.css'
document.getElementById('foo').className = classes.red
```
- 自定义导出命名规则
```
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

```ts
import { defineConfig } from 'vite'

export default defineConfig({
    envPrefix: 'ENV_', // 配置VITE注入客户端变量的前缀,
    css: {
        // 对css的行为进行配置
        // modules配置最终会丢给postcss modules
        modules: {
            // 是对css模块化的默认行为进行覆盖
            localsConvention: 'camelCase', // 修改生成的配置对象的key的展示形式(驼峰还是中划线形式)
            scopeBehaviour: 'local', // 配置当前的模块化行为是模块化还是全局化 (有hash就是开启了模块化的一个标志, 因为他可以保证产生不同的hash值来控制我们的样式类名不被覆盖)
            generateScopedName: '[name]_[local]_[hash:5]',
            // generateScopedName: (name, filename, css) => {
            //     // name -> 代表的是你此刻css文件中的类名
            //     // filename -> 是你当前css文件的绝对路径
            //     // css -> 给的就是你当前样式
            //     console.log('name', name, 'filename', filename, 'css', css) // 这一行会输出在哪？？？ 输出在node
            //     // 配置成函数以后, 返回值就决定了他最终显示的类型
            //     return `${name}_${Math.random().toString(36).substr(3, 8)}`
            // },
            // hashPrefix: "hello", // 生成hash会根据类名 + 一些其他的字符串(文件名 + 他内部随机生成一个字符串)去进行生成, 如果想要生成hash更加的独特一点, 可以配置hashPrefix, 配置的这个字符串会参与到最终的hash生成, （hash: 只要字符串有一个字不一样, 那么生成的hash就完全不一样, 但是只要字符串完全一样, 生成的hash就会一样）
            globalModulePaths: ["./componentB.module.css"], // 代表不想参与到css模块化的路径
        }
    }
})
```
## 5. 环境变量
`Vite`在一个特殊的`import.meta.env`对象上暴露环境变量，这里有一些在所有情况下都可以使用的内建变量
- `import.meta.env.MODE`：`string`应用运行的模式
- `import.meta.env.BASE_URL`：`string`部署应用时的基本`URL`。他由`base`配置项决定
- `import.meta.env.PROD` ：`boolean`应用是否运行在生产环境
- `import.meta.env.DEV`：`boolean` 应用是否运行在开发环境 (永远与`import.meta.env.PROD`相反)
- `import.meta.env.SSR`：`boolean`应用是否运行在`server`

`Vite`内置了`dotenv`这个第三方库，`dotenv`会自动读取`.env`文件，`dotenv`从你的环境目录中的下列文件加载额外的环境变量：
- `.env`：所有情况下都会加载
- `.env.[mode]`：只在指定模式下加载

注意
- _npm run dev 会加载 .env 和 .env.development 内的配置_
- _npm run build 会加载 .env 和 .env.production 内的配置_
- _mode 可以通过命令行 --mode 选项来重写_

加载自定义环境变量：
```json
"scripts": {
	"test": "vite --mode test",
	"dev": "vite",
	"build": "vite build"
},
```

```ts
VITE_HI = "1111111111111111111111111111111111"
```
`console.log(' VITE_HI: ',  import.meta.env.VITE_HI);`

更改`.env`默认地址：
```ts
import { defineConfig } from "vite";
export default defineConfig( {
	envDir:"env"
});
```
## 7. 插件

## 8. 整合TypeScript
在一个单纯的vite项目中，`vite`默认是不会提示ts语法错误并阻断其编译的
`npm i vite-plugin-checker -D`安装
```ts
// vite.config.js
import checker from "vite-plugin-checker";
import { defineConfig } from "vite";
export default defineConfig({
	plugins: [
		checker({
			typescript: true,
		}),
	],
});
```

`Vite`仅执行`.ts`文件的转译工作，并不执行任何类型检查。这意味着，即使我们项目内有`Ts`的语法错误，打包也是可以正常进行的
在打包时进行代码检查：
- 在构建脚本中运行 **tsc --noEmit**
```json
// package.json
"scripts": {
	"dev": "vite",
	// 如果ts检查不通过，vite build就不会执行
	"build": "tsc --noEmit && vite build",
},
```
- 对于`.vue`文件，可以安装`vue-tsc`然后运行`vue-tsc --noEmit`来对你的`.vue`文件做类型检查