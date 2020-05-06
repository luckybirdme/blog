---
title: webpack 介绍
date: 2019-12-08 11:28:22
tags: JavaScript
---

> webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

<!-- more -->



## 一. webpack 的由来

#### 1. 打包由来 
- 以前 web 页面开发，把 html,js,css 都写在同一个页面，或者通过标签 script 方式引入。
- 页面多个 script 加载 js 资源会发起多次 http 请求，拖慢页面生成，希望能将多个 js 文件合并到一个，只发起一次 http 请求。
- 多个 js 文件可通过手动拷贝到一个文件，这个过程称为打包；对于 css 文件也是类似的打包过程；但是手动打包非常影响开发效率。
- 于是就有了 webpack 这样的打包工具，一键完成所有 js,css 文件的打包，合并到一个 bundle 文件中。


#### 2. 打包过程

- 始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- 初始化编译器：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
- 确定入口文件：根据配置中的 entry 找出所有的入口文件；
- 开始编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
- 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- 确定出口文件：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。


#### 3. 更多作用
- 随着模块化 (import) 开发，会产生模块之间的共用，需要生成模块间的依赖图，分先后顺序打包文件。
- 对于新特性 (ES6), 新语言 (TypeScript) 的使用，为了兼容不同浏览器，在打包的时候进行预处理。
- 除了文件合并，还能进行代码压缩，无效代码删减，图片转码等，进一步优化页面加载的性能。
- 利用 Code Splitting 特性，实现按需加载或并行加载多个 bundle，可利用并发下载能力，减少首次访问白屏时间。
- 目前主流的框架 (vuejs, reactjs) 的项目工具都采用 webpack 进行开发，一键配置，高效开发。



## 二. webpack 的核心概念

#### 1. 入口 entry
- 指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。
- 可指定一个入口起点或多个入口起点，配置参数 entry。

```js
// webpack.config.js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```


#### 2. 出口 output
- 告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件。
- 可以通过在配置中指定一个 output 字段，来配置这些处理过程。

```js
// webpack.config.js 
module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```


#### 3. loaders
- loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript），比如 TypeScript。
- loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，并添加到依赖图中，然后利用 webpack 的打包能力，对它们进行处理。

##### loaders 主要配置 test 和 use 参数
```js
// webpack.config.js 
const config = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```
##### 常见 loaders 包括

- css-loader 解析 @import 和 url()，并对引用的依赖进行解析。
- style-loader 在 HTML 中注入 script 标签，将 css 添加到 DOM 中。通常与 css-loader 结合使用。
- babel-loader 将 ES2015+ 代码转译为 ES5。
- ts-loader 将 TypeScript 代码转译为 ES5。
- sass-loader 加载 sass/scss 文件并编译成 css。
- html-loader 将 HTML 导出为字符串。
- vue-loader 加载和转译 Vue 组件。
- file-loader 将文件提取到输出目录，并返回相对路径。
- url-loader 和 file-loader 一样，但如果文件小于配置的限制值，可以返回 data URL。



#### 4. plugins
- loader 被用于转换某些类型的模块，而 plugins 在整个生命周期里都可以执行，从定义环境变量到打包优化压缩。
- plugins 目的在于解决 loader 无法实现的其他事，因为 plugins 能够进一步加工多个 Loader 的输出内容。

##### plugins 主要配置

```js
// webpack.config.js 
const HtmlWebpackPlugin = require('html-webpack-plugin'); 

const config = {
  plugins: [
  	// webpack bundle 到单页面应用文件 html ，包括 css,js 等 hash 命令引用 
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```

##### 常见 plugins 包括

- HtmlWebpackPlugin 自动生成入口文件，并将最新的资源注入到 HTML 中。
- DefinePlugin 用以定义在编译时使用的全局常量。
- CommonsChunkPlugin 用以创建独立文件，常用来提取多个模块中的公共模块。
- HotModuleReplacementPlugin 在运行过程中替换、添加或删除模块，而无需重新加载整个页面。
- UglifyjsWebpack Plugin 对 js 进行压缩，减小文件的体积。
- DllPlugin 拆分 bundle 减少不必要的构建。


## 三. webpack 其他特性

#### 1. webpack，与 grunt 和 gulp 有啥不同
- gulp和grunt是流管理工具，通过一个个task配置执行用户需要的功能，如格式检验，代码压缩等，值得一提的是，经过这两者处理的代码只是局部变量名被替换简化，整体并没有发生改变，还是你的代码。
- 而webpack则进行了更彻底的打包处理，更加偏向对模块语法规则进行转换。主要任务是突破浏览器的鸿沟，将原本浏览器不能识别的规范和各种各样的静态文件进行分析，压缩，合并，打包，最后生成浏览器支持的代码，因此，webapck打包过后的代码已经不是你写的代码了，或许你再去看，已经看不懂啦。


#### 2. 如何利用 webpack 来优化前端性能
- 压缩代码。删除多余的代码、注释、简化代码的写法等等方式。可以利用webpack的UglifyJsPlugin和ParallelUglifyPlugin来压缩JS文件， 利用cssnano（css-loader?minimize）来压缩css。
- 删除死代码（Tree Shaking）。将代码中永远不会走到的片段删除掉。可以通过在启动webpack时追加参数--optimize-minimize来实现
- 利用CDN加速。在构建过程中，将引用的静态资源路径修改为CDN上对应的路径。可以利用webpack对于output参数和各loader的publicPath参数来修改资源路径。
- 提取公共代码，避免重复打包。
- 按需加载代码，缩短首屏加载时间。


#### 3. 如何利用 webpack 生成多页面应用
- 单页应用可以理解为webpack的标准模式，直接在entry中指定单页应用的入口即可，这里不再赘述。
- 多页应用的话，可以使用webpack的 AutoWebPlugin来完成简单自动化的构建，但是前提是项目的目录结构必须遵守他预设的规范。

#### 4. 如何在 vue 项目中实现按需加载
- 采用 webpack 的  require.ensure()
- es提案的import() (推荐)

```js
{   
	path: '/promisedemo',   
	name: 'PromiseDemo',   
	component: resolve => require.ensure([], () => resolve(require('../components/PromiseDemo')), 'demo')
}, {   
	path: '/hello',   
	name: 'Hello',   
	// component: Hello   
	component: resolve => require.ensure([], () => resolve(require('../components/Hello')), 'demo')
}

// 下面2行代码，没有指定webpackChunkName，每个组件打包成一个js文件。 
const ImportFuncDemo1 = () => import('../components/ImportFuncDemo1') 
const ImportFuncDemo2 = () => import('../components/ImportFuncDemo2')
export default new Router({   
	routes: [ 
		{      
			path: '/importfuncdemo1',      
			name: 'ImportFuncDemo1',       
			component: ImportFuncDemo1     
		},    
		{       
			path: '/importfuncdemo2',       
			name: 'ImportFuncDemo2',       
			component: ImportFuncDemo2   
		}   
	] 
})
```


#### 5. 如何提高 webpack 的构建速度
- 多入口情况下，使用CommonsChunkPlugin来提取公共代码
通过externals配置来提取常用库
- 利用DllPlugin和DllReferencePlugin预编译资源模块 通过DllPlugin来对那些我们引用但是绝对不会修改的npm包来进行预编译，再通过DllReferencePlugin将预编译的模块加载进来。
- 使用 Happypack 实现多线程加速编译
- 使用webpack-uglify-parallel来提升uglifyPlugin的压缩速度。 原理上webpack-uglify-parallel采用了多核并行压缩来提升压缩速度
- 使用Tree-shaking和Scope Hoisting来剔除多余代码。

#### 6. webpack 的热更新是如何做到的
- webpack的热更新又称热替换（Hot Module Replacement），缩写为HMR。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。 
- 在 webpack 的 watch 模式下，文件系统中某一个文件发生修改，webpack 监听到文件变化，根据配置文件对模块重新编译打包，并将打包后的代码通过简单的 JavaScript 对象保存在内存中。
- 通过 sockjs（webpack-dev-server 的依赖）在浏览器端和服务端之间建立一个 websocket 长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端，同时也包括第三步中 Server 监听静态文件变化的信息。浏览器端根据这些 socket 消息进行不同的操作。当然服务端传递的最主要信息还是新模块的 hash 值，后面的步骤根据这一 hash 值来进行模块热替换。
- HotModuleReplacement.runtime 是客户端 HMR 的中枢，它接收到上一步传递给他的新模块的 hash 值，它通过 JsonpMainTemplate.runtime 向 server 端发送 Ajax 请求，服务端返回一个 json，该 json 包含了所有要更新的模块的 hash 值，获取到更新列表后，该模块再次通过 jsonp 请求，获取到最新的模块代码。


#### 7. webpack3 和 webpack4 的区别
- 新增了mode参数来表示是开发还是生产，production 侧重于打包后的文件大小，development侧重于goujiansud。
- 移除 loaders 参数，必须使用rules（在3版本的时候loaders和rules 是共存的但是到4的时候只允许使用rules）。
- 移除了CommonsChunkPlugin (提取公共代码)，用optimization.splitChunks和optimization.runtimeChunk来代替。
- 升级happypack插件（happypack可以进行多线程加速打包）。
- 支持es6的方式导入JSON文件，并且可以过滤无用的代码。


#### 8. babel-loader 编译 ES6 代码到 ES5 的原理
- 解析（Parsing）： babel 通过词法和语法解析，把代码解析成抽象语法树 AST (Abstract Syntax Tree)。
- 转换（Transformation）： 对特定的语法进行抽象树的转换，比如 ES6 的箭头函数。
- 转换（Transformation）： 通过 polyfill 的插件，对某些语法进行兼容性转换。
- 生成（Code Generation）: 最后把抽象语法树生成 babel-generator 符合 ES5 规范的代码。 