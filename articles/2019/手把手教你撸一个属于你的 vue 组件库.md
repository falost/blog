> 作者：falost
> 
> 地址：https://falost.cc/article/5de3a11a5b7bf34a057353d3

> 生命不止，折腾不止，技术的成长在于爬坑的多少！ ——— 我说的

好久不见，甚是想念！

### 前言

在日常工作中，我们经常会使用别人开发的组件，比如：element、iView、Mint、Ant Design Vue 等 UI 组件库。
使我们的开发效率提升数倍（别人写好的，拿来就用）平常我们也会开发一些好用的组件，以供自己的项目使用，但都是自己通过相对或绝对路径引入使用的，那么我们如何做到像上面的组件库一样，可以通过下面的几种方式引用呢：


```javascript
import Fui from 'fui'
```

```javascript
const Fui = require('fui')
```

或CND引入

```html
<script src="../fui.js"></script>
```

```javascript
Vue.use(Fui)
```

或多个组件按需加载，减少项目体积,需要借助 `babel-plugin-component`

```javascript
import { Banner, Search } from 'fui'
Vue.component(Banner.name, Banner)
Vue.component(Search.name, Search)
```

### 代码实现

我们需要构建一个 vue 的项目，我们可以使用 vue 的脚手架提供的模版 webpack-simple 来完成

```bash
vue init webpack-simple <project-name>
```

在生成了我们所需要的项目基础构建后，我们先来了解下项目目录结构
```
├── README.md
├── build
│   ├── config.js // 公共 js 配置
│   ├── utils.js
│   ├── vue-loader.conf.js // vue-loader 配置文件
│   ├── webpack.common.js // 全局打包配置
│   ├── webpack.component.js // 单个文件打包配置
│   ├── webpack.config.js // 开发运行配置
│   └── webpack.fui.js // 全局样式基础配置
├── components.json // 所有组件的清单文件
├── dist // 打包生成存放目录
├── example // cdn 引入案例展示目录
│   └── index.html
├── index.html // 入口文件
├── package.json
├── packages // 组件库
├   ├──index.js // 所有组件的入口
│   └── search // 单个组件的存放目录
│       ├── index.js // 组件的入口文件
│       └── src // 组件实现代码目录
│           └── main.vue // 组件模版文件
├── src
│   ├── App.vue
│   ├── assets
│   ├── main.js
│   ├── mixins
│   │   └── datas.js // 为单个组件生成基础数据所使用的
│   ├── scss // 样式配置目录
│   │   ├── _common-style-color.scss
│   │   ├── _common-style.scss
│   │   ├── _common_fun.scss
│   │   ├── _mixins.scss
│   │   ├── fui.scss
│   │   └── index.js
│   └── utils // 公共代码库
│       ├── md5.js
│       ├── session.js
│       └── tools.js
└── types // typescrip 声明文件目录
    ├── component.d.ts
    ├── components
    │   └── banner.d.ts
    ├── fui.d.ts
    └── index.d.ts
```
这里，我对脚手架生成的结构做了调整，增加了 build 目录，专门来放置打包脚本的

另外这里只需要重点关注下打包脚本和 package 目录即可。
下面的配置，我沿用了些之前项目的配置

来看看脚手架的内容吧,我就直接贴代码了！

先看下开发模式配置所需要的

> build/webpack.config.js

```javascript
/*
 * @Descripttion: webpack 配置文件
 * @version:  1.0.0
 * @Author: falost
 * @Date: 2019-10-17 15:34:22
 * @LastEditors: falost
 * @LastEditTime: 2019-10-22 19:50:37
 */
const path = require('path')
const webpack = require('webpack')
const Components = require('../components.json')

const utils = require('./utils')
const config = require('./config')
const vueLoaderConfig = require('./vue-loader.conf')

const HtmlWebpackPlugin = require('html-webpack-plugin')

const NODE_ENV = process.env.NODE_ENV
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
const createLintingRule = () => ({
  test: /\.(js|vue)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  include: [resolve('src'), resolve('test')],
  options: {
    formatter: require('eslint-friendly-formatter'),
    emitWarning: !config.dev.showEslintErrorsInOverlay
  }
})
module.exports = {
  context: path.resolve(__dirname, '../'),
  entry: NODE_ENV == 'development' ? './src/main.js' : Components,
  output: {
    path: path.resolve(process.cwd(), './dist'),
    publicPath: NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath,
    filename: NODE_ENV == 'production' ? '[name]/index.js' : '[name].js',
    chunkFilename: '[id].js',
    // libraryTarget: 'commonjs2',
    libraryTarget: 'umd',
    library: 'fui-[name]',
    umdNamedDefine: true // 会对 UMD 的构建过程中的 AMD 模块进行命名。否则就使用匿名的 define
  },
  module: {
    rules: [
      ...(config.dev.useEslint ? [createLintingRule()] : []),
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'),resolve('packages'), resolve('node_modules/webpack-dev-server/client')],
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  resolve: {
    alias: config.alias,
    extensions: ['*', '.js', '.vue', '.json'],
    modules: ['node_modules']
  },
  devServer: {
    historyApiFallback: true,
    noInfo: true,
    overlay: true
  },
  performance: {
    hints: false
  },
  devtool: '',
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true,
    })
  ]
}
```

单个组件打包所需配置文件，该配置会生成一个一个的组件文件：`fui.banner.js`

> build/webpack.component.js

```javascript
/*
 * @Descripttion: webpack 配置文件
 * @version:  1.0.0
 * @Author: falost
 * @Date: 2019-10-17 15:34:22
 * @LastEditors: falost
 * @LastEditTime: 2019-10-31 14:28:33
 */
const path = require('path')
const webpack = require('webpack')
const Components = require('../components.json')

const utils = require('./utils')
const config = require('./config')
const vueLoaderConfig = require('./vue-loader.conf')
const ProgressBarPlugin = require('progress-bar-webpack-plugin');
// const VueLoaderPlugin = require('vue-loader/lib/plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')

const NODE_ENV = process.env.NODE_ENV
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  context: path.resolve(__dirname, '../'),
  entry: Components,
  output: {
    path: path.resolve(process.cwd(), './dist'),
    publicPath: config.build.assetsPublicPath,
    filename: 'static/js/[name].js',
    chunkFilename: '[id].js',
    // libraryTarget: 'commonjs2',
    libraryTarget: 'umd',
    library: 'fui-[name]',
    umdNamedDefine: true // 会对 UMD 的构建过程中的 AMD 模块进行命名。否则就使用匿名的 define
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'),resolve('packages'), resolve('node_modules/webpack-dev-server/client')],
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  resolve: {
    alias: config.alias,
    extensions: ['*', '.js', '.vue', '.json'],
    modules: ['node_modules']
  },
  performance: {
    hints: false
  },
  devtool: '',
  plugins: [
    // new HtmlWebpackPlugin({
    //   filename: 'index.html',
    //   template: 'index.html',
    //   inject: true,
    // }),
    new webpack.optimize.UglifyJsPlugin({
      sourceMap: true,
      compress: {
        warnings: false,
        dead_code: process.env.NODE_ENV === 'development' ? false : true , // 移除没被引用的代码
        drop_debugger: process.env.NODE_ENV === 'development' ? false : true, // 移除项目中的debugeer
        drop_console: process.env.NODE_ENV === 'development' ? false : true, // 移除console.*的方法
        collapse_vars: process.env.NODE_ENV === 'development' ? false : true, // 内嵌定义了但是只用到一次的变量
        reduce_vars: process.env.NODE_ENV === 'development' ? false : true,// 提取出出现多次但是没有定义成变量去引用的
      }
    }),
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].css'),
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`, 
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      allChunks: true,
    }),
    new ProgressBarPlugin(),
    // new VueLoaderPlugin()
  ]
}

if (process.env.NODE_ENV === 'production') {
  module.exports.devtool = ''
  // http://vue-loader.vuejs.org/en/workflow/production.html
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.LoaderOptionsPlugin({
      minimize: true
    })
  ])
}

```

再看看全部组件打包所需要的配置代码

> build/webpack.common.js

```javascript
/*
 * @Descripttion: 整体组件包
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-10-22 18:37:05
 * @LastEditors: falost
 * @LastEditTime: 2019-10-31 14:42:31
 */
const path = require('path')
const webpack = require('webpack')
const ProgressBarPlugin = require('progress-bar-webpack-plugin')

const ExtractTextPlugin = require('extract-text-webpack-plugin')
const utils = require('./utils')
const config = require('./config')
const vueLoaderConfig = require('./vue-loader.conf')

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  entry: {
    app: [resolve('packages')]
  },
  output: {
    path: path.resolve(process.cwd(), './dist'),
    publicPath: config.build.assetsPublicPath,
    filename: 'fui.common.js',
    chunkFilename: '[id].js',
    libraryExport: 'default',
    library: 'FUI',
    libraryTarget: 'umd'
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: config.alias,
    modules: ['node_modules']
  },
  performance: {
    hints: false
  },
  stats: {
    children: false
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'),resolve('packages'), resolve('node_modules/webpack-dev-server/client')],
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  plugins: [
    new ProgressBarPlugin(),
    new webpack.optimize.UglifyJsPlugin({
      sourceMap: true,
      compress: {
        warnings: false,
        dead_code: true , // 移除没被引用的代码
        drop_debugger: true, // 移除项目中的debugeer
        drop_console: true, // 移除console.*的方法
        collapse_vars: true, // 内嵌定义了但是只用到一次的变量
        reduce_vars: true,// 提取出出现多次但是没有定义成变量去引用的
      }
    }), 
    new webpack.optimize.UglifyJsPlugin({ // js、css都会压缩
      mangle: {
          except: ['$super', '$', 'exports', 'require', 'module', '_']
      },
      compress: {
          warnings: false
      },
      output: {
          comments: false,
      }
    }),
    new ExtractTextPlugin({
      filename: 'fui-common.css',
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`, 
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      allChunks: false,
    }),
    // new VueLoaderPlugin()
  ]
};

```

在三个文件当中，分别引入了 `config.js、utils.js、vue-loader.conf.js`,这三个文件主要是基础配置和打包样式需要的配置，不过在 vue-loader 配置中增加了sass 全局变量等内容，可以根据需求来添加，并不是刚需。

再附上 package.json 所需要的脚本命令吧
```json
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot --config build/webpack.config.js ",
    "build": "npm run build:common && npm run build:component",
    "build:common": "cross-env NODE_ENV=production webpack --progress --hide-modules --config build/webpack.common.js",
    "build:fui": "webpack --progress --hide-modules --config build/webpack.fui.js",
    "build:component": "cross-env NODE_ENV=production webpack --progress --hide-modules --config build/webpack.component.js"
}
```
基础的准备工作基本已经做完了，下来就看看业务组件的配置吧！

首先将我们的项目运行起来

```
npm run dev
```

我们在 package 目录中创建我们需要组件文件夹吧，这里我创建了一个 search 组件，目录结构如下：
```
├── packages
│   └── search
│       ├── index.js
│       └── src
│           └── main.vue

来分别看看组件的代码实现，和我们平常实现方式相同

> packages/search/index.js

```javascript
/*
 * @Descripttion: 搜索组件模块入口
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-10-24 18:49:53
 * @LastEditors: falost
 * @LastEditTime: 2019-10-24 18:51:58
 */
import Search from './src/main'

if (typeof window !== 'undefined' && window.Vue) {
  window.Vue.component('fui-search', Search)
}
/* istanbul ignore next */
Search.install = function (Vue) {
  Vue.component(Search.name, Search)
}

export default Search
export { Search }

```
> packages/search/src/main.vue

```html
<!--
 * @Descripttion: 搜索组件
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-10-24 18:50:25
 * @LastEditors: falost
 * @LastEditTime: 2019-11-25 18:29:42
-->
<template>
  <div class="fui-search" ref="search">
    <div class="fui-input-box">
      <input type="text" ref="searchInput" v-model="searchKeyWords" :focus="!showPlaceholder" class="fui-input" @focus="searchEvent" @change="searchEvent" @blur="searchEvent" >
      <div class="fui-search-placeholder fui-flex-center" @click="clickPlaceholder" v-show="showPlaceholder">
        <span class="icon iconfont icon-chazhao fui-flex-center"></span>
        <span class="fui-placeholder fui-fs16">{{ placeholder }}</span>
      </div>
      <span v-show="searchKeyWords && searchKeyWords.length > 0" class="fui-search-btn icon iconfont icon-chazhao fui-flex-center" @click="searchEvent"></span>
    </div>
  </div>
</template>

<script>
import { datas } from '@/mixins/datas'
export default {
  name: 'fui-search',
  data () {
    return {
      searchKeyWords: '',
      showPlaceholder: true,
      placeholder: '搜索课程名称'
    }
  },
  // 基础数据混入 按需配置即可
  mixins: [ datas ],
  // 传入组件的参数
  props: {
    data: {
      type: Object,
      default () {
        return {}
      }
    }
  },
  methods: {
    /**组件所需要的方法类集合**/
  }
}
</script>

<style lang="scss">
  /**组件样式**/
</style>

```

这就是一个组件的实现代码

那么，我们在来看看打包和使用吧

首先配置下这两个文件即可

```
├── components.json
├── packages
│   ├── index.js
``

> components.json

```json
{
  "search": "./packages/search/index.js"
}
```

> package/index.js

```javascript
/*
 * @Descripttion: 暴露所有模块
 * @version: 1.0.0
 * @Author: falost
 * @Date: 2019-10-21 20:25:57
 * @LastEditors: falost
 * @LastEditTime: 2019-11-15 21:02:04
 */
import Search from './search'

const components = {
  Search
}
const install = function (Vue, opts = {}) {
  for (const key in components) {
    if (components.hasOwnProperty(key)) {
      Vue.component(components[key].name, components[key])
    }
  }
}

/* istanbul ignore if */
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}

export default {
  version: '1.0.0',
  install,
  ...components
}

```
到这里，我们一个简单的组件就已经完成了，接下来，就是需要我们来测试和打包使用了。

测试的话，就和平常我们使用其他组件一样引入既可以

> src/main.js

```javascript
import package from '../packages'
Vue.use(packages)
```
如果执行了`npm run build` 之后，会分别生成一个全局的组件包，fui.common.js 和单个组件的 js 文件，这里样式和代码是分开的，需要分别引入。

关于如何发布到 npm 当中，会在下一篇文章中说明。

本文到此结束，感谢您的耐心阅读！

文中如有不正之处，欢迎留言指教！