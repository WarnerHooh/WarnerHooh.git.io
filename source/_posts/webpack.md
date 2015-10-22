title: Frontend pack tool -- Webpack(refers)
date: 2015-06-06 15:12:52
tags: [Tools, Webpack]
category: Tools
---

# webpack #

## 1.webpack是什么 ##
webpack是一个模块打包工具(module bundler),它可以管理模块间的依赖关系,模块可以是js,也可以是css,text等其它静态资源

## 2.webpack的特点 ##
1. 可以将依赖关系树拆分成块并且动态加载
2. 保证初始化的加载时间
3. 所有的静态资源都很容易的被当做一个模块
4. 可以方便的引用第三方的类库作为模块
5. 针对每一个模块,可管理配置的api很多
6. 适合大型的单网页应用项目
7. 丰富的插件,不用在功能需求时还要借助第三方,本身的插件库足够面对足够多的场景

## 3.安装步骤 ##
1. 安装node.js

2. 通过npm安装webpack
` npm install webpack -g `
现在webpack已经安装到全局,你也可以通过命令行全局的使用webpack命令了

3. 在你自己的Project里使用webpack

你最好在自己的项目中在通过npm安装一个本地webpack:
` npm install webpack --save-dev `


## 4.如何使用webpack Step by Step ##
### Hello World ###
先按照以下文件目录结构创建

*entry.js*
```javascript
//随便写一行输出代码
document.write("It works.");
```

*index.html*
```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        // 在这里,我们没有直接引用*entry.js*,而是引用了*bundule.js*,原因是*bundule.js*
        // 就是经过webpack编译后的js
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

然后在命令行中输入webpack命令
` webpack ./entry.js bundle.js `

它将会编译你的*entry.js*文件,编译后的文件名为*bundle.js*
如果你的控制台信息输入如下内容,就说明你的第一次webpack成功了!

```
Version: webpack 1.7.2
Time: 154ms
    Asset  Size  Chunks             Chunk Names
bundle.js  1437       0  [emitted]  main
chunk    {0} bundle.js (main) 28 [rendered]
    [0] ./tutorials/getting-started/setup-compilation/entry.js 28 {0} [built]
```

### 增加一个js的引用 ###
然后我们在上述的文件结构基础上,
在与 entry.js 和 index.html 的平层目录中再添加一个新的js文件

*content.js*
```javascript
// CommonJS的模块化写法,与node.js的模块化一致
module.exports = "It works from content.js.";
```

然后我们修改* entry.js *文件
```javascript
// document.write("It works."); 去掉直接输出内容

// 通过require模块的方式加载content.js模块
document.write(require("./content.js"));
```

猜猜如果运行 webpack 将要发生什么? webpack 会将 *entry.js* 中依赖的 *content.js* 模块打包成一个文件,
那么我们运行一下:

` webpack ./entry.js bundle.js `

刷新你的浏览器,我们可以看到页面中输入了 It works.

### 第一个Loader ###

我们在使用 webpack 时, 不仅仅是希望可以将js文件模块化并且打包构建, 对于web中常用的其它格式的文件也有模块化的需求, 比如css等.
但是 webpack 的初始化仅提供了对js文件的支持, 所以我们就需要一个 **css-loader** 来处理.

那么首先我们先来安装这个插件,在你的工程根目录下( 常规下你的node_modules目录会直接放置在根目录下 ),运行
` npm install css-loader style-loader `

来看下我们的目录结构是

----entry.js
----content.js
----index.html
----bundle.js

然后我们添加一个新的css文件

-----style.css
```css
body {
    background: yellow;
}
```

接下来修改 *entry.js*, 记得我们在 *entry.js* 中直接require了一个模块, 现在我们把css文件也当做一个模块来引用

```javascript
require("!style!css!./style.css");
document.write(require("./content.js"));
```

最后运行

` webpack ./entry.js bundle.js `
如果你看到你的页面的背景色变成了黄色,那么说明it works. webpack做了什么, 它把css与js一起打包进了模块, 而css是通过js操作内联样式插入到页面dom当中去的

### 我们需要一个配置文件 ###

我们不想每次使用webpack时还在命令行后面缀上一大堆参数, 需要一个配置文件来保存这些参数, webpack的配置文件名就是 *webpack.config.js*

```javascript
module.exports = {
    entry: "./entry.js", //入口文件,需要打包那些文件是从现在入口文件进行分析的
    output: {
        path: __dirname,  //node内置环境变量,当前Project的根目录的路径
        filename: "bundle.js" //打包后的文件, 一般页面中是引用打包后的文件 
    },
    module: {
        // 各种处理器的声明,这里只使用到了css处理器
        // 如上所见, 它的作用就是解析css文件并且和依赖它的模块打包构建成一体
        // 并且通过内联样式的方式嵌入到dom当中
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

现在,你可以直接运行 webpack 了 (当然, 你的webpack.config.js 最好直接被放置在项目的根路径下). 至此,你已经步入了webpack的大门, 接下来我们针对一些具体的业务场景来举例如何使用webpack

### 抽离公用代码 ###

项目中有一个很常见的场景就是, 多个模块间实际是有很多依赖公用模块的, 比如A模块依赖 jquery 和 underscore, B模块也依赖jquery 和 underscore , 如果打包 模块A 和 模块B 时, 那么公用的 jquery 和 underscore 都会和 A,B 被打进一个文件. 由于类库的文件都比较大, 而且类库模块基本不会发生变化, 版本功能迭代更新的大多也都是业务代码, 所以直接打包在一起弊端多多, 分开管理 还可以更 合理的控制缓存, 没有必要刷新未变动的类库文件的缓存.

那么怎么才能将 jquery 和 underscore 这种类库文件单独打包进一个文件, 并且其它模块 在 ` require('jquery') ` 或者 ` require('underscore') `时不会再重复打包呢?

```javascript
module.exports = {
  entry: {
    app: "./app.js",
    vendor: ["jquery", "underscore", ...],
  },
  output: {
    filename: "bundle.js"
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"vendor.bundle.js")
  ]
};
```

我们分析上述的配置文件, 在entry 中我们不再像之前 直接写入一个入口js文件, 而是传入了一个对象, 对象的属性名称就是 你自由配置的包模块名称, 你的包模块可以只包含一个模块, 比如app模包块,只包含了app.js模块, 也可以是多个模块的集合, 比如vender包模块, 它将 jquery 以及 underscore 模块一起构建到其中.

当然,光在 entry 中配置多个入口js 是不行的, 我们注意到在配置文件中多了一个 plugins 参数, 并且引入了一个CommonsChunkPlugin的插件, 望文生义, 它的作用就是 把某个 包模块 编译为一个 公用模块, 使用的方式就是传入包模块的名称: vender 作为第一个参数, 而被编译打包后的文件名作为第二个参数 传入Plugin即可.
那么现在, 你的html文件或者模板文件中引入js的代码就要相应发生变化, 要引入两个js文件

```html
<script src="vendor.bundle.js"></script> <!-- 公用模块 -->
<script src="bundle.js"></script>   <!-- 业务模块 -->
```

*vendor.bundle.js* 就是 jquery 以及 underscore 打包在一起的文件, 而 bundle.js 就是你的 app模块, 另外说明一点的是: app模块打包后的文件里不一定只有一个模块, 相反它只是一个入口模块, 也就是它里面可能也会require其它模块, 那么在webpack打包构建会都会打入一个文件中, 再说明一点的就是: 但是它不会将 jquery 以及 underscore 重复打包进来, 因为咱们已经通过 new webpack.optimize.CommonsChunkPlugin 做了公用模块的配置(如上demo所示).

上述只是针对官网给出的demo做出的翻译 并简单实现了一种业务需求的打包, 原版官网指南请参考 [http://webpack.github.io/docs/tutorials/getting-started/](http://webpack.github.io/docs/tutorials/getting-started/)


## 5.OPTIMIZATION ##
### Minimize ###
打包构建最基础的功能就是 去除缩进, 回车, 空格, 注释, 混淆简化变量函数名等等, webpack也不例外, 而使用的方式就是通过扩展插件的方式, 具体见配置文件

```javascript
module.exports = {
    plugins: [
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            }
        })
    ]
};
```

更多Deduplication, Chunks等Opt配置请参考官网说明
[http://webpack.github.io/docs/optimization.html](http://webpack.github.io/docs/optimization.html)

## 6.自动化 ##

我们在使用webpack时候发现,每次某个js文件发生变化时, 都要重新在命令行下 执行一遍 webpack, 如果持续以这种方式开发 估计一天的代码开发下来 满脑子都是 webpack 这几个字母了, 所以我们能否寻求一个 可以自动watch 模块的变化, 并且自动 跑一遍 webpack的工具呢? 答案是肯定的, 而且肯定的背后是多种自动化的工具, 例如 grunt 和 gulp 等, grunt 流行有一段时间了,但是由于使用时要进行一堆繁琐的配置而被诟病, gulp 是继 grunt 之后流行的一个新的自动化构建工具, 而grunt的缺点被它完美的填平, 所以现在 gulp下的用户粉丝更多一些, 同样的 插件功能也很齐全. 当然, 作为我们此次的需求, 只需要一个watch 文件变化的功能即可. 首先, 我们需要安装 gulp, 具体详见官网
[https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md)

你要做的也很简单, 只需要
` npm install --global gulp `

然后在你本地的 Project中单独安装
` npm install --save-dev gulp `

光安装了 gulp 还不行, 我们需要为 gulp 扩展一个支持 webpack的包, so 接下来:
` npm install --save-dev gulp-webpack-build `

接着修改的*gulpfile.js* 的配置文件

```javascript
'use strict';

var path = require('path'),
    gulp = require('gulp'),
    webpack = require('gulp-webpack-build');

// 你需要定制修改的地方
var src = 'app',  // 需要监测的文件的路径
    dest = 'built', // webpack打包后发布的地址
    webpackOptions = {
        debug: true,
        devtool: '#source-map',
        watchDelay: 200
    },
    webpackConfig = {
        useMemoryFs: true,
        progress: true
    },
    CONFIG_FILENAME = webpack.config.CONFIG_FILENAME;

gulp.task('watch', function() {
    gulp.watch(path.join(src, '**/*.*')).on('change', function(event) {
        if (event.type === 'changed') {
            gulp.src(event.path, { base: path.resolve(src) })
                .pipe(webpack.closest(CONFIG_FILENAME))
                .pipe(webpack.configure(webpackConfig))
                .pipe(webpack.overrides(webpackOptions))
                .pipe(webpack.watch(function(err, stats) {
                    gulp.src(this.path, { base: this.base })
                        .pipe(webpack.proxy(err, stats))
                        .pipe(webpack.format({
                            verbose: true,
                            version: false
                        }))
                        .pipe(gulp.dest(dest));
                }));
        }
    });
});
```

最后, 当我们在命令行执行
` gulp watch `

就发现, 每次当你 ` ctrl + s ` 保存一个文件后, 你的 webpack 会自动的构建一次, 而且 webpack 会优化你的构建性能和 时间开销, 保证你的开发效率.

更多的webpack的高级用法请访问官网 [http://webpack.github.io/](http://webpack.github.io/)