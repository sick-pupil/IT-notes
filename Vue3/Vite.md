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

## 4. CSS Module

## 5. 静态资源

## 6. 插件

## 7. 整合TypeScript
