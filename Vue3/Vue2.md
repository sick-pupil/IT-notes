## 1. Vue-cli-server
### 1. vue.config.js
```json
const AutoImport = require('unplugin-auto-import/webpack')
const Components = require('unplugin-vue-components/webpack')
const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')

const path = require('path');
const resolve = dir => path.join(__dirname, dir);

const isProduction = process.env.NODE_ENV === 'production';
const isDevelopment = process.env.NODE_ENV === 'development';

module.exports = {
  // 基于部署应用包时的基本URL
  publicPath: '/',
  productionSourceMap: false,
  configureWebpack: config => {
    // elementplus按需引入
    const plugins = [
      AutoImport({
        resolvers: [ElementPlusResolver()],
      }),
      Components({
        resolvers: [ElementPlusResolver()],
      })
    ];
    config.plugins.push(...plugins);
    // 为生产环境修改配置...
    if (process.env.NODE_ENV === 'production') {
      config.mode = 'production';
      // 打包文件大小配置
      config.performance = {
        maxEntrypointSize: 10000000,
        maxAssetSize: 30000000
      }
    }
  },
  chainWebpack: config => {
    // 添加别名
    config.resolve.alias
      .set('@', resolve('src'))
      .set('@apis', resolve('src/apis'))
      .set('@assets', resolve('src/assets'))
      .set('@scss', resolve('src/assets/scss'))
      .set('@components', resolve('src/components'))
      .set('@mixins', resolve('src/mixins'))
      .set('@plugins', resolve('src/plugins'))
      .set('@router', resolve('src/router'))
      .set('@store', resolve('src/store'))
      .set('@utils', resolve('src/utils'))
      .set('@config', resolve('src/config'))
      .set('@views', resolve('src/views'));

    // code splitting
    config.optimization.splitChunks({
      cacheGroups: {
        // 第三方组件
        vendors: {
          // 指定chunks名称
          name: `chunk-vendors`,
          // 符合组的要求就给构建
          test: /[\\/]node_modules[\\/]/,
          // 优先级：数字越大优先级越高，因为默认值为0，所以自定义的一般是负数形式,决定cacheGroups中相同条件下每个组执行的优先顺序。
          priority: -10,
          // 仅限于最初依赖的第三方
          chunks: 'initial'
        },
        // 公共组件
        common: {
          name: `chunk-common`,
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          //这个的作用是当前的chunk如果包含了从main里面分离出来的模块，则重用这个模块，这样的问题是会影响chunk的名称。
          reuseExistingChunk: true
        },
        // // 将elementplus单独拆分出来
        element: {
          name: `element-plus`,
          test: /element-plus/,
          priority: 20,
          chunks: 'all'
        },
        // 将vue相关的单独拆分出来
        vue: {
          name: `vue`,
          test: /vue/,
          priority: 20,
          chunks: 'initial'
        }
      }
    });

    // 资源配置：在生产环境和开发环境配上title
    if (isProduction || isDevelopment) {
      config.plugin('html')
        .tap(args => {
          args[0].title = 'test';
          return args;
        });
    }

    // 打包分析
    if (isProduction) {
      if (process.env.npm_config_report) {
        config
          .plugin('webpack-bundle-analyzer')
          .use(require('webpack-bundle-analyzer').BundleAnalyzerPlugin)
          .end();
        config.plugins.delete('prefetch');
      }
    }
  },
  css: {
    // 是否使用css分离插件 ExtractTextPlugin
    // 是否将组件中的 CSS 提取至一个独立的 CSS 文件中 (而不是动态注入到 JavaScript 中的 inline 代码)。
    extract: true,
    // 开启 CSS source maps? 
    sourceMap: false,
    loaderOptions: {
      scss: {
        // 向全局sass样式传入共享的全局变量, $src可以配置图片cdn前缀
        // 详情: https://cli.vuejs.org/guide/css.html#passing-options-to-pre-processor-loaders
        prependData: `
          @import '@scss/variables.scss';
          @import '@scss/mixins.scss';
          @import '@scss/function.scss';
          $src: '${process.env.VUE_APP_BASE_API}';`
      }
    }
  },
  devServer: {
    // 让浏览器 overlay 同时显示警告和错误
    overlay: {
      warnings: true,
      errors: true
    },
    // open: false, // 是否打开浏览器
    host: '',
    port: '8180', // 代理端口
    // https: false,
    hotOnly: true, // 热更新
    proxy: {
      '/ins': {
        target: 'http://pre-fundbackend.jinfuzi.cn/',
      },
      '/alc': {
        target: 'http://10.3.20.64:8090/'
        // target: 'http://192.168.172.113:8090/'
      },
      '/sys': {
        target: 'http://192.168.20.120:27730'
      }
    }
  },
  // 构建时开启多进程处理 babel 编译
  parallel: require('os').cpus().length > 1,
  // https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa
  pwa: {},
  // 第三方插件配置
  pluginOptions: {}
}
```