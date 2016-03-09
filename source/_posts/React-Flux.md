title: React Flux
date: 2015-07-02 23:14:14
tags: [React, Flux, MVC]
category: Javascript
---
# React Flux #

## What's React Flux ##
> Flux is more of a pattern than a framework, and does not have any hard dependencies.

简单的说Flux就是一种前端MV*解决方案(javascript不像Java等后端代码严格独立出Model-View-Controller，一般是指Model和View层的分离)。然而，React声称传统的MVC模式实际上存在很大的弊端，并且不利于大型项目管理，所以FB大牛发明了一种号称优于MVC模式的解决方案 —— Flux。

关于FB给出的官方解释，众说纷纭，各位看官可以仔细琢磨一番。以下是Flux的流程图：
![Flux data flow] (https://github.com/facebook/flux/raw/master/docs/img/flux-diagram-white-background.png)

## How React Flux works ##
在React Flux中有三个重要的概念：*Dispatcher*、*Stores*、*Views*。

- Dispatcher 分发中心。Dispatcher是Flux的中枢，管理着所有数据流动作。它本质上是Store的回调函数，每个Store管理着自己的数据模型，同时通过Dispatcher注册了一个回调函数，当Dispatcher响应Action时将会挂载Action的数据并发送给对应的Store处理。
<!-- more -->
- Stores 数据处理中心。每个独立的模块应该独立出一个Store，Store就是每个模块数据的存储、加工的地方。Store中定义一个供监听数据改变的事件供React最外层模块监听并作出响应(setState)。同时每个Store挂载了对Dispatcher的分发规则(通过actionType匹配)

- Acions 行为动作管理。Action即管理用户的动作并交给Dispatcher分发出去，分发时挂载动作触发时的数据。

- Views React视图模块。View就是React的component模块，通过state变化达到更新视图模板的作用。在Flux中，底层的component都通过上一层的props通信，最外层component通过监听Store中定义的事件动态刷新传入子component的data数据。要在子component上监听事件，将数据封装好后调用Action处理。

## How to start Flux ##
你可以查看[官方example](https://github.com/facebook/flux/tree/master/examples/flux-todomvc)。在看完官方给出的TODO Example后，自己撸了一个简单的Msg Demo，可能更容易理解一点。下面给出代码片段和说明:

``` javascript
目录结构：
+ flux-msg
	+ dist
		- bundle.js
	+ node-modules
	+ src
		+ actions
			- AppAction.js
		+ components
			- MsgInput.jsx
			- MsgList.jsx
			- MsgMain.jsx
		+ constants
			- AppConstant.js
		+ dispatcher
			- AppDispatcher.js
		+ stores
			- AppStore.js
		- app.js
	- Gulpfile.js
	- index.html
	- package.json
```

- 首先我们看下需要用到的npm包*package.json*
```javascript
	{
	  "name": "flux-msg",
	  "version": "0.0.0",
	  "private": true,
	  "devDependencies": {
	    "babel-jest": "^5.2.0",
	    "browserify": "^9.0.3",
	    "del": "^1.1.1",
	    "gulp": "^3.8.11",
	    "gulp-babel": "^5.1.0",
	    "gulp-replace": "^0.5.3",
	    "jest-cli": "^0.4.3",
	    "run-sequence": "^1.0.2",
	    "vinyl-source-stream": "^1.0.0",
	    "flux": "^2.0.1",
	    "keymirror": "~0.1.0",
	    "object-assign": "^1.0.0",
	    "react": "^0.12.0",
	    "envify": "^3.0.0",
	    "reactify": "^0.15.2",
	    "uglify-js": "~2.4.15",
	    "watchify": "^2.1.1"
	  }
	}
```
其中有几个包简单介绍一下，因为在官方TODO中有用到，对于没用过这些包的我来说有些疑惑，路要一步一步走，把不懂的先弄懂了再开始下一步。
[keymirror](https://github.com/STRML/keyMirror)是一个构造常量(更像enum)对象的工具。
```javascript
	var keymirror = require('keymirror');
	keymirror({
		KEY1: null,
		KEY2: null,
		KEY3: null
	})
	==> {KEY1: "KEY1", KEY2: "KEY2", KEY3: "KEY3"}
```
[object-assign](https://github.com/sindresorhus/object-assign)是一个扩展对象的工具，类似$.extend()。
```javascript
	var objectAssign = require('object-assign');

	objectAssign({foo: 0}, {bar: 1});
	==> {foo: 0, bar: 1}

	objectAssign({foo: 0}, {bar: 1}, {baz: 2});
	==> {foo: 0, bar: 1, baz: 2}
```
[envents]()是Nodejs中的内置模块，通过browserify可以在浏览器中使用。它是实现自定义事件和自定义触发事件的工具。
```javascript
	var util = require('util');
	var EventEmitter = require('events').EventEmitter;

	function MyEvent() {
		EventEmitter.call(this);
	}

	util.inherits(MyEvent, EventEmitter);

	MyEvent.prototype.hello = function() {
		console.log('hello function');
		this.emit('test');
	}

	var me = new MyEvent();

	me.on('test', function(data) {
		console.log('test event: ' + data);
	})

	me.hello('xxo');
	//==> hello function
	//==> test event: xxo
```

- Gulpfile.js
这里只是粗略的写了一个gulp脚本
```javascript
var gulp = require('gulp');
var browserify = require('browserify');
var reactify = require('reactify');
var source = require('vinyl-source-stream');
gulp.task('build', function() {
	return browserify({entries: ['./src/app.jsx']})
			.transform('reactify')
			.bundle()
			.pipe(source('bundle.js'))
			.pipe(gulp.dest('./dist'));
});
```

- AppDispatcher.js
```javascript
//直接引用flux
var Dispatcher = require('flux').Dispatcher;
module.exports = new Dispatcher();
```

- AppConstant.js
```javascript
//常量方便维护
var keyMirror = require('keyMirror');
module.exports = keyMirror({
	ADD: null,
	DELETE: null,
	UPDATE: null
});
```

- AppStore.js
该文件中主要功能:①提供对Msg数据的增删改查方法②定义MsgStore，提供对change事件的监听方法和手动触发③将Action方法通过Store的回调函数注册
```javascript
var AppDispatcher = require("../dispatcher/AppDispatcher");
var AppConstant = require("../constants/AppConstant");
var EventEmitter = require('events').EventEmitter;
var assign = require('object-assign');
var CHANGE = 'change';
//actually msg model
var _msgs = [];
function getMsgs() {
	return _msgs;
}
//add
function addMsg(data) {
	_msgs.push(data);
}
//delete
function deleMsg(index) {
	_msgs.splice(index, 1);
}
//update
function updateMsg(index, data) {
	_msgs.splice(index, 1, data);
}
//define MsgStore
var MsgStore = {};
MsgStore = assign({}, EventEmitter.prototype, {
	//emit change event
	emitChange: function() {
		this.emit(CHANGE);
	},

	getAll: function() {
		return getMsgs();
	},

	addChangeListener: function(callback) {
		this.on(CHANGE, callback)
	},

	removeChangeListener: function(callback) {
		this.removeListener(CHANGE, callback);
	}
});
AppDispatcher.register(function(action) {
	var text;

	switch(action.actionType) {
		case AppConstant.ADD:
			text = action.text;
			addTodo(text);
			MsgStore.emitChange();
			break;

		case AppConstant.UPDATE:
			text = action.text;
			updateTodo(action.index, text);
			MsgStore.emitChange();
			break;

		case AppConstant.DELETE:
			deleTodo(action.index);
			MsgStore.emitChange();
			break;

		default:
	}
});
module.exports = MsgStore;
```
- AppAction.js
//该文件中主要提供了针对用户的动作的响应，具体通过AppDispatcher将data挂载到Store中处理并触发change事件
```javascript
var AppDispatcher = require('../dispatcher/AppDispatcher');
var AppConstant = require('../constants/AppConstant');
var AppAction = {
	addMsg: function(data) {
		Dispatcher.dispatch({
			actionType: AppConstant.ADD,
			text: data
		})
	},
	delMsg: function(index) {
		Dispatcher.dispatch({
			actionType: AppConstant.DELETE,
			index: index
		});
	},
	updMsg: function(index, data) {
		Dispatcher.dispatch({
			actionType: AppConstant.UPDATE,
			index: index,
			text: data
		});
	}
}
module.exports = AppAction;
```

- MsgInput.jsx
```javascript
//msg输入组件
var React = require('react');
var TodoAction = require('../actions/AppAction.js');
var MsgInput = React.createClass({
	displayName: 'MsgInput',
	submitHandle: function(e) {
		//代理click在父级元素上，判断触发事件的DOM
		if(e.target === this.refs.submit.getDOMNode()) {
			var date = new Date();
			var o = {
				title: this.refs.title.getDOMNode().value,
				author: this.refs.author.getDOMNode().value,
				content: this.refs.content.getDOMNode().value,
				date: date.getFullYear() + '-' + (date.getMonth()+1) + '-' + date.getDate()
			}
			
			AppAction.addMsg(o);
		}
	},
	render: function() {
		return (
			<div onClick={this.submitHandle}>
				<div className="msgRow" data-asd="xxx">
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
```

- MsgList.jsx
```javascript
//msg列表组件
var React = require('react');
var MsgLists = React.createClass({
	displayName: 'MsgList',
	getInitialState: function() {
		return{
			msgs : [
				{title: 'asdasdsd', content: 'xxxxx', author: 'warner', date: '2015-06-18'},
				{title: 'asdasdsd', content: 'xx', author: 'warner', date: '2015-06-18'},
				{title: 'asdasdsd', content: 'x', author: 'warner12354', date: '2015-06-18'}
			]
		}
	},
	render: function() {
		//通过父组件的props传入data数据
		var msgs = this.props.msgs;
		var lists = msgs.map(function(msg, index) {
			return (
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
```

- MsgMain.jsx
```javascript
var React = require('react');
var MsgList = require('./MsgList.jsx');
var MsgInput = require('./MsgInput.jsx');
var AppStore = require('../stores/AppStore.js');
//通过store中的getAll方法获取所有msgs数据
function getMsgs() {
	return {
		msgs: AppStore.getAll()
	}
}
var MsgMain = React.createClass({
	getInitialState: function() {
		return getMsgs();
	},
	//当组件渲染完成后监听store的change事件，触发时调用_changeFunc
	componentDidMount: function() {
		AppStore.addChangeListener(this._changeFunc);
	},
	//当组件将被卸载时取消监听store的change事件
	componnetWillUnmount: function() {
		AppStore.removeChangeListener(this._changeFunc);
	},
	render: function() {
		return (
			<div>
				//将获取的所有msgs数据对象通过props传到子组件
				<MsgList msgs={this.state.msgs} />
				<MsgInput />
			</div>
		)
	},
	//当其他子组件click等事件触发时
	  -->调用AppAction中的对应方法
	  -->AppDispatcher挂载AppAction数据并对事件进行分发
	  -->在Appstore中找到dispatcher注册的相对应的回调函数并执行
	  -->手动触发change事件*emitChange()*
	  -->另一个子组件中通过监听的change事件回调函数*_changeFunc*，setState()更新数据流
	  -->更新传入到子组件的props属性
	  -->整个View层渲染完成
	_changeFunc: function() {
		this.setState(getMsgs());
	}
});
module.exports = MsgMain;
```

- app.jsx
```javascript
//调用React方法渲染组件
var React = require('react');
var MsgMain = require('./components/MsgMain.jsx');
React.render(<MsgMain />, document.getElementById('msg'));
```

- index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<div id="msg"></div>
	<script type="text/javascript" src="dist/bundle.js"></script>
</body>
</html>
```

ok，这个简单的DEMO就完成了，在MsgInput组件中输入的数据在经过Flux处理后会更新到MsgList组件中渲染出来。这个功能很简单，主要是为了理清Flux的原理和思路。

## What Else: PubSub ##
我的理解是Flux的作用就是完成子模块之间的通信，本质就是通过注册回调函数实现的，只是中间绕了几道弯，抽离出来的Action、Dispatcher、Store等都是为了让整个流程更清晰，更利于大型应用的维护。不知道个人的理解是否有误，其实在我学习Flux之前，完成过一个React的小应用，当时也存在组件之间通信的问题，当时是通过PubSub订阅发布的方式实现的，我想这两者之间应该有异曲同工之妙吧...