title: React with Webpack
date: 2015-06-21 10:38:07
tags: ["webpack", "react"]
category: Javascript
---

#React with Webpack#

## React ##
React是facebook团队打造的前端js框架，现在很多人都在使用React构造自己的前端项目，可见React已经非常成熟了，而且React的生态系统也非常完善。之前在segment fault和tui cool上拜读了很多关于React的文章，有幸能在新项目中来运用React。最开始就参照别人的demo写了一些简单的模块，懵懵懂懂也并没有理解React的优越性。在摸索了一段时间之后，发现React使用起来真是得心应手啊，自己也试着写一点对于React的总结：

### React 使前端代码模块化 ###

React认为一切皆模块，一段js代码是一个模块，一个css样式文件是一个模块。这样一个较复杂的项目就可以分成很多个完全独立的模块，每个同事并行开发，然后再将这些模块组装好。
React支持也推荐使用CommonJs规范编写代码，使用require加载一个模块，使用exports对外暴露接口。如果你写过nodejs应该对这种语法非常熟悉。
``` javascript

	var lib = require('lib.js'); //加载对于lib模块的依赖
	lib.method(); //然后就可以调用lib模块的方法

	//_innerFun属于内部方法，对外是不可见的
	function _innerFun(){}

	function funA(){
		_innerFun();
	}

	function funB(){}

	//对外暴露的模块接口调用
	module.exports = function() {
		//仅暴露需要的方法，其他均视为私有属性
		return {
			methodA: funA,
			methodB: funB
		}
	}
```

### React 使我们改变传统的js编码思路 ###
我们经常吐槽js代码不好维护，别说是别人写的代码了，就连我们自己写的代码在一段时间之后也感觉读不懂了(就算写了一堆注释)！一部分原因是代码组织不够清晰，另一方面由于我们平时都需要使用jQuery操作DOM，代码中大部分逻辑都是和DOM产生了复杂的关系。而且别人要是稍微改了一下DOM结构，我们的代码逻辑可能就需要重新组织一次了。
React推荐不要直接操作DOM，需要将DOM与数据操作逻辑解耦合。使用jsx语法(将js代码和html代码组织在一起，内部解析机制是，遇到`<tagName>`就通过html语法解析，遇到`{expression}`就通过javascript语法解析)，和我们用过的模板引擎有些相似。需要注意的是：
> 关键字冲突

由于html和javascript写在一起，难免会有语法冲突。比如*class*、*for*等既是html属性又是javascript关键字。所以React将这些特殊html用特殊的关键字代替，`class => className`，`for => htmlFor`更多用法请参考React API

> 事件绑定

React事件绑定参考原始的在DOM上写事件监听的方法。比如`<a href="#" onClick={this.clickFunc}>Click</a>` 这里我们就在某个DOM上绑定了一个我们自定义的clickFunc方法。①React推荐事件代理的方式去实现事件监听，我们尽量把事件监听写在父级容器上再作分发。②在clickFunc中我们从默认的arguments中拿到的event是React包装后的事件对象，常用的事件属性都可以访问，但是如果你需要访问原始event对象可以通过event.nativeEvent得到。

> 语法规范

在jsx语法中，并不是像我们平时写html代码那么随意。①DOM属性有限制，比如我们需要在一个DOM上加上一个myAttr属性，我们想这样写`<div myAttr="myVal"></div>`，然而通过React渲染出来的div却把myAttr丢掉了，因为React认为这不是一个合法的写法，我们可以这样写`<div data-myAttr="myVal"></div>`，也就是说，React推荐使用*data-*作为自定义属性的规范。②标签嵌套要规范。我们平时写html代码时可能会忽略标签的嵌套规则，比如`<p className="outerP">asdasdsa<p className="innerP">xxxxx</p></p>`，在P标签中嵌套了另一个P标签，这种写法被React解析出来会是怎么样的呢？`<p class="outerP"><span>asdasdsa</span></p>  <p class="innerP">xxxxx</p>   <p></p>`，咦，asdasdsa被包裹了一层span。但是……真是哔了个狗了，原本的嵌套关系不存在了，而且还多出了一个空的P标签是什么鬼？使用div嵌套可破之~所以我们嵌套html标签时务必注意规则，避免产生不必要的bug。

### React 神秘的虚拟DOM ###
为什么React这么火？因为React的虚拟DOM机制：通过state与DOM分离，并非数据一改变就会重新渲染整个DOM结构，React内部有一个复杂的diff算法，它会计算出真正需要改变的DOM元素，保留不变的DOM渲染真正改变的DOM，很大程度上提高了页面渲染效率，在PC上可能效果不够明显，这在移动端是非常有必要的！

### React 和 Angular ###
很多人把React和Angular(1.x)对比(我承认我也曾努力寻找这方面的答案)，其实他们的宗旨不一样，谈不上孰好孰坏，下面我们看一下两者的区别。

> Angular追求的是一种大而全的概念；React则渴望成为小而精

  Angular致力于MVVM(Model - View - ViewModel)的javascript框架，Angular提供了包括$scope、路由(Route)、控制器(Controller)、服务(Service)、指令(Directive)的明确MV*开发思路(如果你使用过后端开发代码，你会发现我们现在使用javascript代码分离简直烂的一比，和JAVA的Spring MVC相比，javascript一直处于一个垃圾堆一样的环境)，Angular正是出于将垃圾分类的目的耀世而生。

  其实和Angular相比React都算不上一个javascript框架，我想React仅仅是作为一种前端模块化的解决方案。要和Angular比较的话，React应该只属于Angualr的View层。React推崇模块化理念，在React中一切资源(js代码、css代码)皆模块。React支持AMD、CommonJs、ES6的模块化语法规范，相比AMD个人更偏爱CommonJS语法，因为它的语法更清晰明了。如果你写过JAVA代码，那么你对*import*一定不陌生，在JAVA代码中，class之间完全独立，如果要想引用其他class，就需要import其他class，这样代码的依赖关系一目了然。

  题外话：前一段时间还在纠结该使用seajs还是requirejs对于前端代码的模块化，听说seajs理念先进，按需加载，可是打包和兼容老代码麻烦；requirejs先把模块全部加载完全，生态系统和兼容性好，但配置又太麻烦。现在使用Webpack + React或者Gulp + Browserify一切迎刃而解，简直不能太爽！

> Angular的双向绑定 vs React的状态机

  要说Angualr1.x和React各自的看家本领，那么Angualr就是data-bind对于数据的双向绑定，React则是state状态机映射的虚拟DOM。

  Angualr的ViewModel对于Model和View是双向的，也就是说可以在Model层改变ViewModel达到更改View渲染的目的，也可以再View层改变ViewModel达到更改Model的目的。这样我们不需要关心数据的同步更新，我们只需要明确在Controller、Service、Directive所操作的是同一个ViewModel。然而这样的双向绑定在Angular内部通过脏检查实现，对性能开销特别大，而且一个ViewModel只要发生改变，用于渲染整个ViewModel的View层就需要完全刷新，这对于开发手机WEB应用而且数据量比较大时是致命的！听说在Angular2.x将采用完全区别于1.x版本的机制，而且这个性能问题也会得到完美解决，让我们拭目以待！

  React内部有两个比较特别的属性：props和state。props是Render时传递进来的，props应该像常量一样是不可变的。state是用于模块内部渲染使用的状态机，一旦状态机发生改变，React就会计算出新的DOM结构并渲染出来，也可以理解为state对于view的单向绑定。虽然没有Angular的双向绑定，但是React的虚拟DOM足够强大，比如一个state存放了10000条的list，我们改变了其中一条，那么页面上仅会对这一条发生改变的数据绑定DOM作出刷新，其他9999则会保持不变。这也正是我们所需要的！

> Angular想得更周到；React更灵活

  Angualr内部提供了很多$service，而且采用依赖注入的方式调用。比如$http、$filter、$location、$compile等很方便的调用，而React除了React还是React，连ajax也没有实现。当然React会显得更灵活，模块之间的通信都要自己实现，想怎么写怎么写……一般我们采用订阅发布方式(PubSub)实现模块之间的通信。值得一提的是React的Mixins能够实现类似JAVA的面向切面编程。

### React 之Hello world ###
> 首先我们需要在index.html中加入对React库的引用

`<script type="text/javascript" src ="react.js"></script>`
`<script type="text/javascript" src ="JSXTransformer.js"></script>`


> 我们在index.html中创建一个container

`<div id="helloContainer"></div>`

> 编写HelloWorld模块

````javascript
//注意type是text/jsx
<javascript type="text/jsx">
//注意首字母大写
var HelloWorld = React.createClass({
	getInitialState: function() {
		return {msg: "hello world"}
	},
	render: function() {
		var state = this.state, //state作为内部逻辑渲染数据模型，发生改变实时刷新DOM
			props = this.props; //props作为外部传进来的数据，一般不可变
		return(
			//注意return必须返回一个根节点，也就是不能返回几个并列的兄弟节点
			<div className="helloDiv">
				<p>{ state.msg }</p>
				<p>this is props.name</p>
			</div>
		)
	}
}
});
</javascript>
````

> 调用React渲染 Done!

//通过标签形式渲染模块 | 选中页面中的某个DOM元素作为模块的加载容器
`React.render(<HelloWorld data={name: "Hooh"}></HelloWorld>, document.getElementById('#helloContainer'));`
还是比Angular简单易上手，只是jsx语法可能一开始会有点不适应

## Webpack ##
从上面的例子我们可以看到加载了*JSXTransformer.js*，这是用来解析jsx语法的，也就是我们所写的jsx代码并不能够直接被浏览器执行，需要先借助JSXTransformer解析成javascript代码供浏览器执行。一种更好的解决方案是在后端对jsx代码解析成普通的javascript代码，页面直接引用解析后的javascript代码，那么你需要借助webpack来实现了。当然，这仅仅是一个切入点。webpack的强大之处太多了，让我数数……总之开始webpack就对了。那么我们开始一个进阶的栗子:

### 树形结构 ###
````
----app
	----lib
		----react.js
		----jquery.js
		----pubsub.js
	----src
		----css
			----global.css
			----msg.css
		----img
		----js
			----module
				----MsgList.jsx
				----MsgInput.jsx
			----main.jsx
			----common.jsx
	----build
		----common.bundle.js
		----main.bundle.js
	----index.html
````

lib中存放了用到的库文件
src中存放开发环境的代码模块
build中存放打包输出的上线环境文件
index.html静态页面

### webpack.config.js webpack配置文件 ###
````javascript
var webpack = require('webpack');
var path = require("path");
module.exports = {
	resolveLoader: { root: path.join(__dirname, "node_modules") }, //指定loader路径
    context: __dirname + path.sep + "app", //指定自定义配置路径
    //入口文件
	entry: {
		//common.jsx作为公用文件
		common: './src/js/common.jsx',
		//main.jsx作为app入口文件
		main: './src/js/main.jsx'
	},
	//打包输出文件
	output: {
		//打包到build文件夹下
		path: path.join(__dirname, "build"),
		//以[name].bundle.js命名
		filename: "[name].bundle.js"
	},
	module: {
		//通过何种loader解析不同的文件
		loaders: [
			{ test: /\.jsx$/, loader: 'jsx-loader'},
            { test: /\.css$/, loader: "style-loader!css-loader"}, //多个loader通过!连接
		]
	},
	//一些插件
	plugins: [
		//打包公共的common.bundle.js文件，以后对于这些公用文件的引用不会重复打包
		new webpack.optimize.CommonsChunkPlugin("common", "./common.bundle.js"),
		//压缩js文件
		new webpack.optimize.UglifyJsPlugin({
			compress: {
				warnings: false
			}
		})
	],
	//解析配置
	resolve: {
		//在模块中require以下文件时可忽略后缀名
		extentions: ['', 'js', 'jsx', 'css'],
		//配置别名[其他模块对于以下文件的引用可以直接require]
		alias: {
			react: __dirname + "/app/lib/react/react-with-addons.js",
			pubsub: __dirname + "/app/lib/pubsub-js/src/pubsub.js",
			zepto: __dirname + "/app/lib/zepto/zepto.js"
		}
	}
}

````

### 代码片段 ###
> index.html 入口页面

````html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<div class="msgContainer">
		<div id="msgList"></div>
		<div id="msgInput"></div>
	</div>

	<script type="text/javascript" src="build/common.bundle.js"></script>
	<script type="text/javascript" src="build/main.bundle.js"></script>
</body>
</html>
````

> common.jsx 公共文件

````javascript
//在weback配置文件中配置了别名，可直接引用
require("react");
require("zepto");
require("pubsub");
//未配置别名的文件通过相对路径引用
require("../css/global.css");

````

> main.jsx 入口js文件

````javascript
var React = require("react");

var MsgList = require("./components/MsgList.jsx");
var MsgInput = require("./components/MsgInput.jsx");

function submit(msg) {
	//TODO Ajax submit
}

React.render(<MsgList></MsgList>, document.getElementById('msgList'));
React.render(<MsgInput data={submit: submit}></MsgInput>, document.getElementById('msgInput'));
````

> MsgList.jsx msg列表模块

````javascript
require('../../css/msg.css');
var React = require('react');
var MsgLists = React.createClass({
	displayName: 'MsgList',
	//初始化时的state数据
	getInitialState: function() {
		return{
			msgs : [
				{title: 'asdasdsd', content: 'xxxxx', author: 'warner', date: '2015-06-18'},
				{title: 'asdasdsd', content: 'xx', author: 'warner', date: '2015-06-18'},
				{title: 'asdasdsd', content: 'x', author: 'warner', date: '2015-06-18'}
			]
		}
	},
	render: function() {
		var lists = this.state.msgs.map(function(msg, index) {
			return (
				//调用map输出时推荐在每个iterator上添加key属性
				<li key={"msgli-" + index}>
					<span className="msgIndex">{index + 1}</span>
					<span className="msgTitle">{msg.title}</span>
					<span className="msgContent">{msg.content}</span>
					<span className="msgAuthor">{msg.author}</span>
					<span className="msgDate">{msg.date}</span>
				</li>
			);
		});
		return (
			<ul className="msgLists">
				{ lists }
			</ul>
		)
	}
});

module.exports = MsgLists;
````

> MsgInput.jsx msg输入模块

````javascript
require('../../css/msg.css');
var React = require('react');
var MsgInput = React.createClass({
	displayName: 'MsgInput',
	submitHandle: function(e) {
		//代理click在父级元素上，判断触发事件的DOM
		if(e.target === this.refs.submit.getDOMNode()) {
			var date = new Date();
			var o = {
				//通过this.refs获取模块中的DOM索引。getDOMNode()获取对应的原生DOM
				title: this.refs.title.getDOMNode().value,
				author: this.refs.author.getDOMNode().value,
				content: this.refs.content.getDOMNode().value,
				date: date.getFullYear() + '-' + (date.getMonth()+1) + '-' + date.getDate()
			}
			//调用props的submit方法
			this.props.data.submit.call(this, o);
		}
	},
	render: function() {
		return (
			//click事件方法监听
			<div onClick={this.submitHandle}>
				<div className="msgRow">
					<label htmlFor="author">Author</label>
					<input type="text" ref="author" id="author" />
				</div>
				<div className="msgRow">
					<label htmlFor="title">Title</label>
					<input type="text" ref="title" id="title" />
				</div>
				<div className="msgRow">
					<label htmlFor="content">Content</label>
					<textarea ref="content" id="content"></textarea>
				</div>
				<button ref="submit">Submit</button>
			</div>
		)
	}
});

module.exports = MsgInput;
````

### webpack 使用技巧 ###
> *webpack* 执行一次开发模式的编译

> *webpack -p* 执行一次发布环境(production)的编译，会压缩代码(minification)

> *webpack -d* 生成SourceMaps

> *webpack --watch* 无需任何配置，监听到文件改变时(真正require到的文件)会自动编译，而且速度非常快！