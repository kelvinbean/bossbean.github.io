---
title: 优化webpack打包速度
---

最近发现项目越做越大的时候，webpack的打包速度越来越慢。虽然是现在又出现了一个号称比webpack打包快，并且零配置的东西，叫parcel。

但是目前来看，感觉parcel还是在摸索阶段。我也确实尝试了一下，打包确实挺快的。但是不知道会有什么别的意料之外的问题，而且vue-cli目前用的还是webpack打包，所以我还是说说怎么提升一下webpack的打包吧。

首先打包速度慢，我们还是要分析一下打包的状况吧。


### webpack-bundle-analyzer

webpack有个webpack-bundle-analyzer的工具。可以用来分析打包出来的状况。

其实vue-cli已经给配置并且安装了这个工具，所以不用再自己去安装与配置。

<!-- More -->

webpack.prod.conf.js里面有着相关配置。

```bash
var config = require('../config')

...//此处省略一万字

if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
```

这里可以看到config.build.bundleAnalyzerReport如果为true，那么就会执行这个分析工具。很明显默认为false.

顺着上面的内容找到了config目录下的index.js文件。

ok,发现此处的bundleAnalyzerReport，这一项的赋值是process.env.npm_config_report这个。

```bash
build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../dist/index.html'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './',//kelvin修改的
    productionSourceMap: false,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
```

所以我们在package.json文件的scripts加入新的命令。用来方便我们以后分析时直接用

``` bash
"analyz": "set NODE_ENV=production && set npm_config_report=true && npm run build",
```

网上很多的资料都是省去了set，估计是版本不一样的问题，我直接写会报错。
这里的命令就是设置当前的环境是生产环境，并且设置npm_config_report为true，然后执行build.

就这样。执行以下命令我们调出了分析的页面。

```bash
npm run analyz
```

你的分析页面大概长这样。

![Image text](/images/img/webpack1.png)

上面是我的现在优化后的图片。显示块越大，代表合出来的文件越大。你可以看出一些端疑。

优化前，我也分析出，我之前使用的echarts文件还有都被合到了多个文件里面，而且文件都特别大。这些代码还是一样的。这样的事情我肯定不能忍啦。

另外我还发现我使用的ueditor也被打包进去了。然而我并不需要打包它。因这种种原因，我们就分析出了打包那么慢的原因了。


### 设置不打包ueditor

先解决这个比较简单的。

看过我之前写的那篇[编写ueditor的文章](http://www.bossbean.com/2018/01/25/vue%E9%A1%B9%E7%9B%AE%E5%BC%95%E5%85%A5ueditor/)就知道。


我在写editor组件的时候，才引入这些ueditor组件的。因此，删掉editor组件里面引用ueditor插件的那些import。

把它放到根目录下的index.html里。以script的形式引入。当然使用cdn的话就更好了。

```bash
        <script src="./static/plugins/utf8-jsp/ueditor.config.js"></script>
        <script src="./static/plugins/utf8-jsp/ueditor.all.min.js"></script>
        <script src="./static/plugins/utf8-jsp/lang/zh-cn/zh-cn.js"></script>
        <script src="./static/plugins/utf8-jsp/ueditor.parse.min.js"></script>
```


因为ueditor会设置一个全局变量UE，那么为了防止打包的时候报错，我们需要再build文件夹里module.exports上加上。

```bash
     externals:{
        UE:'UE'
      },
```

官网文档解释的很清楚，就是webpack可以不处理应用的某些依赖库，使用externals配置后，依旧可以在代码中通过CMD、AMD或者window/global全局的方式访问。

so，没有压缩这个编辑器，速度提升了。

### DllPlugin 和 DllReferencePlugin

DllPlugin 可以把我们需要打包的第三方库打包成一个 js 文件和一个 json 文件，这个 json 文件中会映射每个打包的模块地址和 id，DllReferencePlugin 通过读取这个json文件来使用打包的这些模块。

接下来，解决echarts的问题。

看过我之前关于[echarts的文章](http://www.bossbean.com/2017/12/29/%E5%9C%A8vue%E9%87%8C%E4%BD%BF%E7%94%A8echarts/)，就可以知道我们在使用的vue-echarts-v3，已经是封装成了vue的组件了。所以每次引入都就会把这些相同的代码打包进去。这样打包出来的文件很大，同时也影响速度。


在 build 文件夹中新建 webpack.dll.conf.js

```bash
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    vendor: ['vue/dist/vue.common.js','vue-router', 'babel-polyfill','axios','vue-echarts-v3','vue-ba','vue-uweb']
  },
  output: {
    path: path.join(__dirname, '../static/js'),
    filename: '[name].dll.js',
    library: '[name]_library'       // vendor.dll.js中暴露出的全局变量名
  },
  plugins: [
    new webpack.DllPlugin({
      path: path.join(__dirname, '.', '[name]-manifest.json'),
      name: '[name]_library'
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
};
```

这里把我项目需要用到的第三方库我都写了进去。

然后在 package.json 中再加个配置命令，也是方便我们以后使用。

```bash
"scripts": {
    ...
    "build:dll": "webpack --config build/webpack.dll.conf.js"
}
```

执行 npm run build:dll 命令来生成 vendor.dll.js 和 vendor-manifest.json

需要在 index.html 引入 vendor.dll.js

```bash
    <script src="./static/js/vendor.dll.js"></script>
```

vendor-manifest.json 文件生成我们不需要管它，在打包的时候DllReferencePlugin会自动去读取。

接下来就在 webpack.base.config.js 中通过 DLLReferencePlugin 来使用 DllPlugin 生成的 DLL Bundle

```bash
var webpack = require('webpack');

module.exports = {
    entry: {
        app: ['./src/main.js']
    },
    module: {
        ...
    }
    // 添加DllReferencePlugin插件
    plugins: [
        new webpack.DllReferencePlugin({
            context: path.resolve(__dirname, '..'),
            manifest: require('./vendor-manifest.json')
        }),
    ]
}
```

就这样大功告成。。我再次使用npm run build。比以前快了70%。

原因是你已经把你需要用的第三方库都打包了，然后打包的时候就不需要再去打包这些库。从而减少了文件的大小，而且也打包速度也变快了。

但是如果你又要引入新的第三方库的话，需要再去webpack.dll.conf.js里面去添加vendor。

然后再去执行一遍npm run build:dll 。生成的新的vendor.dll.js和manifest.json。然后再提交上去。


### 总结

当然还有非常多的方法可以提升webpack打包速度。我这里只是简单的介绍我使用的方法。