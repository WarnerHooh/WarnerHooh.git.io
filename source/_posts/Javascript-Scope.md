title: Javascript Scope
date: 2015-06-06 23:18:57
tags: [Javascript, scope]
category: Javascript
---

# Javascript Scope #
要清楚的了解javascript运行逻辑，必须掌握javascript的作用域。这不仅有助于我们编写健壮的javascript代码，更有利于我们分析产生的代码bug。scope(作用域)、closure(闭包)、this(关键字)、namespace(命名空间)、function scope(函数作用域)、global scope(Global作用域)、lexical scope(词义作用域)、public/private scope(公共/私有作用域)等概念将在本文中介绍到。


## 什么是作用域 ##
在javascript中，scope指的是你代码运行环境的上下文。Scopes可以在全局被定义，也可以在局部被定义。了解javascript的scope有助于提升javascript developer的编码能力。

<!-- more -->

## 什么是全局作用域 ##
在你编写一行javascript代码之前，你当前所处的上下文环境就是Global Scope。如果此时我们申明了一个变量，那么它就是全局变量。
```javascript
// global scope
var name = 'Todd';
```
全局变量是一把双刃剑，要学会使用全局变量并要拿捏得当。你疆场会听到人们说“全局变量实在糟糕，尽量少使用全局变量！”，但是从来没有想过这是为什么。其实全局变量并不糟糕，当你需要创建Modules模块或者APIs时你就必须得使用全局变量来完成了。
每个人都在使用jQuery，Like this ` jQuery('.myClass'); `
我们在Global Scope中使用jQuery，也可以把jQuery理解为scope的namespace命名空间。jQuery是全局对象，同时也是jQuery库中的命名空间，jQuery的所有方法都通过jQuery命名空间进行访问。

## 什么是局部作用域 ##
局部作用域是指除了全局作用域以外定义的作用域，每个方法内部定义的作用域就是一个独立的局部作用域。
如果我在一个方法中定义了一个变量，那么这个变量就处在改方法的局部作用域中。举个栗子：
```javascript
// Scope A: Global scope out here
var myFunction = function () {
  // Scope B: Local scope in here
};
```
除非主动暴露出来，否则我们在全局作用域中是无法获取到局部作用域的。也就是说，如果我们在某个方法内部定义了一个变量，那么它只存在于该方法的局部作用域中，出了这个方法，都访问不到该局部作用域中的变量。
```javascript
var myFunction = function () {
  var name = 'Todd';
  console.log(name); // Todd
};
// Uncaught ReferenceError: name is not defined
console.log(name);
```
上面的变量` name `存在于局部作用域中，且没有暴露在父级作用域中，因此输出了` undefined `

## 函数作用域 ##
在javascript中，所有的scopes都是通过函数作用域创建。作用域不能通过` for `或者` while `等循环表达式创建，也不能通过` if `或者` switch `等条件表达式创建。一个新的函数即一个新的作用域。举个栗子：
```javascript
// Scope A
var myFunction = function () {
  // Scope B
  var myOtherFunction = function () {
    // Scope C
  };
};
```

## 词义作用域 ##
当一个方法处于另一个方法內部时，内部的方法总是能够读取外部方法作用域中的内容，这就叫做词义作用域。举个栗子：
```javascript
// Scope A
var myFunction = function () {
  // Scope B
  var name = 'Todd'; // defined in Scope B
  var myOtherFunction = function () {
    // Scope C: 'name' is accessible here!
  };
};
```
你也许注意到了，在这里`myOtherFunction`只是被定义了但未调用。为了证明上面的阐述，我们来调用定义的方法并通过console输出：
```javascript
var myFunction = function () {
  var name = 'Todd';
  var myOtherFunction = function () {
    console.log('My name is ' + name);
  };
  console.log(name);
  myOtherFunction(); // call function
};

// Will then log out:
// 'Todd'
// 'My name is Todd'
```
词义作用域在平时很实用，当任何变量/对象/方法定义在父级作用域中时，在它的作用域链中都能够访问到：
```javascript
var name = 'Todd';
var scope1 = function () {
  // name is available here
  var scope2 = function () {
    // name is available here too
    var scope3 = function () {
      // name is also available here!
    };
  };
};
```
最重要的是词义作用域不能反向，也就是说内部作用域内容无法在外部作用域中读取到：
```javascript
// name = undefined
var scope1 = function () {
  // name = undefined
  var scope2 = function () {
    // name = undefined
    var scope3 = function () {
      var name = 'Todd'; // locally scoped
    };
  };
};
```

## 作用域链 ##
域链给一个已知的函数建立了作用域。正如我们所知的那样，每一个被定义的函数都有自己的嵌套作用域，同时，任何被定义在其他函数中的函数都有一个本地域连接着外部的函数 - 这种连接被称作链。这就是在代码中定义作用域的地方。当我们在处理一个变量的时候，JavaScript就会开始从最里层的域向外查找直到找到要找的那个变量、对象或函数。

## 闭包 ##
闭包和词法作用域非常相近。一个关于闭包如何工作的更好或者更实际的例子就是返回一个函数的引用。我们可以返回域中的东西，使得它们可以被其父域所用。
```javascript
var sayHello = function (name) {
  var text = 'Hello, ' + name;
  return function () {
    console.log(text);
  };
};
```
我们此处所用的闭包使得sayHello里的域无法被公共域访问到。单是调用这个函数不会发生什么，因为它只是返回了一个函数而已：
```javascript
sayHello('Todd'); // nothing happens, no errors, just silence...
```
这个函数返回了一个函数，就是说它需要分配然后才是调用：
```javascript
var helloTodd = sayHello('Todd');
helloTodd(); // will call the closure and log 'Hello, Todd'
```
好吧，我撒谎了，你可以调用它，或许你已经看到了像这样的函数，但是这会调用你的闭包：
```javascript
sayHello2('Bob')(); // calls the returned function without assignment
```
AngularJS就为其 $compile 方法用了上面的技术，当前作用域作为引用传递给闭包：
` $compile(template)(scope); `
我们可以猜测代码或许应该像下面这样：
```javascript
var $compile = function (template) {
  // some magic stuff here
  // scope is out of scope, though...
  return function (scope) {
    // access to 'template' and 'scope' to do magic with too
  };
};
```
一个函数不是只有返回什么东西的时候才会称作闭包。简单地使词法作用域的外层可以访问其中的变量，这便创建了一个闭包。


## 私有域和公共域 ##

在许多编程语言中，你将听到关于公共域和私有域，在 JavaScript 里没有这样的东西。但是我们可以通过像闭包一样的东西来模拟公共域和私有域。

我们可以通过使用 JavaScript 设计模式比如模块模式，来创建公共域和私有域。一个简单的创建私有域的途径就是把我们的函数包装进一个函数中。如我们之前学到的，函数创建作用域来使其中的东西不可被全局域访问：
```javascript
(function () {
  // private scope inside here
})();
```
我们可能会紧接着创建一个新的函数在我们的应用中使用：
```javascript
(function () {
  var myFunction = function () {
    // do some stuff here
  };
})();
```
当我们准备调用函数的时候，它不应在全局域里：
```javascript
(function () {
  var myFunction = function () {
    // do some stuff here
  };
})();

myFunction(); // Uncaught ReferenceError: myFunction is not defined
```
成功！我们就此创建了一个私有域。但是如果我像让这个函数变成公共的，要怎么做呢？有一个很好的模式（被称作模块模式）允许我们正确地处理函数作用域。这里我在全局命名空间里建立了一个包含我所有相关代码的模块：
```javascript
// define module
var Module = (function () {
  return {
    myMethod: function () {
      console.log('myMethod has been called.');
    }
  };
})();

// call module + methods
Module.myMethod();
```
在这里，return 的东西就是 public 方法返回的东西，它可以被全局域访问。我们的模块来关心我们的命名空间，它可以包含我们想要任意多的方法在里面：
```javascript
// define module
var Module = (function () {
  return {
    myMethod: function () {
    },
    someOtherMethod: function () {
    }
  };
})();

// call module + methods
Module.myMethod();
Module.someOtherMethod();
```
那私有方法呢？这里是很多开发者做错的地方，他们把所有的函数都堆砌在全局域里以至于污染了整个全局命名空间。可工作的函数代码不一定非在全局域里才行，除非像 APIs 这种要在全局域里可以被访问的函数。这里我们来写一个没有被返回出来的函数：
```javascript
var Module = (function () {
  var privateMethod = function () {
  };
  return {
    publicMethod: function () {
    }
  };
})();
```
这就意味着 publicMethod 可以被调用，但是 privateMethod 则不行，因为它被域私有了！这些私有的函数可以是任何你能想到的对象或方法。

但是这里还有个有点拧巴的地儿，那就是任何在同一个域中的东西都可以访问同一域中的其他东西，就算在这儿函数被返回出去以后。也就是说，我们的公共函数可以访问私有函数，所以私有函数依然可以和全局域互动，但是不能被全局域访问。
```javascript
var Module = (function () {
  var privateMethod = function () {
  };
  return {
    publicMethod: function () {
      // has access to 'privateMethod', we can call it:
      // privateMethod();
    }
  };
})();
```
这种互动是充满力量同时又保证了代码安全。JavaScript中很重要的一块就是保证代码的安全，这就解释了为什么我们不能接受把所有的函数都放在公共域中，因为这样的话，他们都被暴露出来很容易受到攻击。

下面有个例子，返回了一个对象，用到了 public 和 private 方法：
```javascript
var Module = (function () {
  var myModule = {};
  var privateMethod = function () {

  };
  myModule.publicMethod = function () {

  };
  myModule.anotherPublicMethod = function () {

  };
  return myModule; // returns the Object with public methods
})();

// usage
Module.publicMethod();
```
比较精巧的命名方式就是在私有方法名字前加下划线，这可以帮我们在视觉上区分公共的和私有的方法：
```javascript
var Module = (function () {
  var _privateMethod = function () {

  };
  var publicMethod = function () {

  };
})();
```
这里我们可以借助面向对象的方式来添加对函数的引用：
```javascript
var Module = (function () {
  var _privateMethod = function () {
  };
  var publicMethod = function () {
  };
  return {
    publicMethod: publicMethod,
    anotherPublicMethod: anotherPublicMethod
  }
})();
```

## Keyword -- **this**  ##
>When a function of an object was called, the object will be passed into the execution context as this value

首先我们需要了解javascript中this的工作原理
JavaScript 有一套完全不同于其它语言的对 this 的处理机制。 在五种不同的情况下，this 指向的各不相同。

1. 全局范围内
	this;
	当在全部范围内使用 this，它将会指向全局对象。
	浏览器中运行的 JavaScript 脚本，这个全局对象是 window。

2. 函数调用
	这里 this 将会指向全局对象，即 window。
	```javascript
	function foo(){
	　　this.x = 1;
	　　alert(this.x);
	}
	test(); // 1
	```
	为了证明this就是全局对象，我对代码做一些改变：
	```javascript
	var x = 1;
	function test(){
	　　alert(this.x);
	}
	test(); // 1
	```
	运行结果还是1。再变一下：
	```
	var x = 1;
	function test(){
	　this.x = 0;
	}
	test();
	alert(x); //0
	```
	ES5 注意: 在严格模式下（strict mode），不存在全局变量。 这种情况下 this 将会是undefined。
	
	```javascript
	var nav = document.querySelector('.nav'); // <nav>
	var toggleNav = function () {
	  console.log(this); // this = <nav> element
	};
	nav.addEventListener('click', toggleNav, false);
	```
	我们在某个对象添加事件监听时，事件回调函数中的this指向仍是事件监听的对象。

	这里还有个问题，就算在同一个函数中，作用域也是会变，this 的值也是会变：
	```javascript
	var nav = document.querySelector('.nav'); // <nav>
	var toggleNav = function () {
	  console.log(this); // <nav> element
	  setTimeout(function () {
	    console.log(this); // [object Window]
	  }, 1000);
	};
	nav.addEventListener('click', toggleNav, false);
	```
	在` setTimeout `代码块中打印出来的` this `改变了，这是为什么呢？其实我们知道` setTimeout `方法的调用应该是` window.setTimeout `当然代码块中的` this `指向方法调用者，也就是` window `！

	如果我们想要访问这个` this `值，有几件事我们可以让我们达到目的。可能以前你就知道了，我们可以用一个像 that 这样的变量来缓存对` this `的引用：
	```javascript
	var nav = document.querySelector('.nav'); // <nav>
	var toggleNav = function () {
	  var that = this;
	  console.log(that); // <nav> element
	  setTimeout(function () {
	    console.log(that); // <nav> element
	  }, 1000);
	};
	nav.addEventListener('click', toggleNav, false);
	```

3. 方法调用内部
	```javascript
	function foo(){
		alert(this.x);
	}
	
	var o = {};
	o.x = 1;
	o.m = foo;
	
	o.m(); // 1
	```
	这个例子中，在foo()方法内部，this 指向调用该方法的对象，也就是 o 对象，所以 ` this.x ` = ` o.x `。

4. 调用构造函数
	所谓构造函数，就是通过这个函数生成一个新对象（object）。这时，this就指这个新对象。
	```javascript
	function test(){
		this.x = 1;
	}
	var o = new test();
	alert(o.x); // 1
	```
	运行结果为1。为了表明这时this不是全局对象，我对代码做一些改变：
	```javascript
	var x = 2;
	function test(){
		this.x = 1;
	}
	var o = new test();
	alert(x); //2
	```
	运行结果为2，表明全局变量x的值根本没变。


5. 显式的设置/改变 this
   call()和apply()是函数对象的一个方法，它的作用是改变函数的调用对象，它的第一个参数就表示改变后的调用这个函数的对象。因此，this指的就是这第一个参数。
   ```javascript
	var x = 0;
	function test(){
		alert(this.x);
	}
	var o={};
	o.x = 1;
	o.m = test;
	o.m.apply(); //0
   ```
   apply()的参数为空时，默认调用全局对象window。因此，这时的运行结果为0，证明this指的是全局对象。如果把最后一行代码修改为
   ```javascript
   o.m.apply(o); //1
   ```
   运行结果就变成了1，证明了这时this已经被显式的改变成了对象o。

	.bind()
	不同于上述方法，使用 .bind() 不会调用一个函数， 它只是在函数运行前绑定了一个值。ECMASCript5 当中才引入这个方法实在是太晚太可惜了，因为它是如此的美妙。如你所知，我们不能出传递参数给函数，就像这样：
	```javascript
	// works
	nav.addEventListener('click', toggleNav, false);

	// will invoke the function immediately
	nav.addEventListener('click', toggleNav(arg1, arg2), false);
	```
	我们可以通过在其中创建一个新的函数来搞定它：
	```javascript
	nav.addEventListener('click', function () {
	  toggleNav(arg1, arg2);
	}, false);
	```
	还是那个问题，这个改变了作用域的同时我们也创建了一个不需要的函数，这对性能是一种浪费如果我们在循环内部绑定事件监听器。 尽管这使得我们可以传递参数进去，似乎应该算是 .bind() 的用武之地，但是这个函数不会被执行：
	```javascript
	nav.addEventListener('click', toggleNav.bind(scope, arg1, arg2), false);
	```
	这个函数不会执行，并且作用域可以根据需要更改，但是参数还是在等待被传入。