title: Object Oriented Javascript
date: 2016-03-09 22:07:22
tags: [Javascript, Object Oriented, inheritance, extend]
category: Javascript
---

# Object Oriented Javascript
## 类
### 如何声明一个类
在ES6之前，我们这样声明一个类

``` javascript
function Vehicle(engines) {
  this.engines = engines;
}

Vehicle.prototype.ignition = function() {
  console.log('Turning on ' + this.engines + ' engines!');
}

var vehicle = new Vehicle(2);

console.log(vehicle.engines);
// 2
vehicle.ignition();
// Turning on 2 engines!
```
当使用`new`操作符调用一个函数(和函数名无关，用作类时习惯首字母大写)时，会使用该函数构造一个对象，大概发生了以下几件事：

- 所以在`Vehicle`函数中，创建一个空对象
- 将该对象绑定到函数调用的`this`
- 将该对象的 *[[Prototype]]* 指向`Vehicle.prototype`
- 如果`Vehicle`函数没有返回其他对象，默认返回该对象

初始化的值如`engines`会分别绑定到实例化的所有对象上，而`prototype`上的值(方法)则会被所有实例化的对象共享。

### 类和对象
类和对象是一种怎样的关系？对象是类的实例。没错，但是这背后有什么故事呢？首先列几个知识点。

1. 类其实就是方法`function`，每个`function`都有一个`prototype`对象，该对象有一个特殊的属性`constructor`指向该`function`。

2. 实例也就是对象`object`，几乎每个`object`都有一个 *[[Prototype]]* 属性的对象，可以通过非标准的接口`__proto__`查看，该对象指向构造该实例的构造函数的原型链。 所以

	``` javascript
	vehicle.__proto__ === Vehicle.prototype;
	// true
	```
	ES6之后新增了标准接口`Object.getPrototypeOf`和`Object.setPrototypeOf`来获取和设置对象的 *[[Prototype]]* 。
	
3. 所有的`function`也是`object`(由`Function`构造)。所以

	```javascript
	Vehicle.__proto__ === Function.prototype
	// true
	```

4. 通过 `instanceof` 判断对象是否是某个类的实例。判断的逻辑是一层一层遍历对象的 *[[Prototype]]* , 如果某个类的`prototype`出现在该对象的 *[[Prototype]]* 链中则返回true。

	``` javascript
	vehicle instanceof Vehicle;
	// true
	
	function Foo() {}
	vehicle instanceof Foo;
	// false
	
	vehicle.__proto__.__proto__ = Foo.prototype;
	vehicle instanceof Foo;
	// true
	```

有图有真相
![alt prototype](./images/javascript-prototype.jpg)

### 如何实现类的继承
实现类的继承方法很多，大概列举以下几种

#### 原型链继承

``` javascript
function Car(wheels) {
  this.wheels = wheels;
}

Car.prototype = new Vehicle(2);

Car.prototype.drive = function() {
  console.log('Steering ' + this.wheels + ' wheels and moving forward!');
}

var car = new Car(4);
console.log(car.engines);
// 2
console.log(car.hasOwnProperty('engines'));
// false
car.ignition();
// Turning on 2 engines!
```
该方案思路是将子类`Car`的原型链指向一个父类`Vehicle`实例对象，这样通过子类实例化的对象`car`就能从原型链上面调用到父类的方法`ignition`了。注意该方案存在以下问题:

① 在实例化子类对象时没办法动态向父类构造函数值传参，就上面的代码我们就无法再构造一个具有4个`engine`的`car`了。

② 由于直接将prototype指向实例化的父类对象，导致初始化的属性被直接绑定到了子类`Car`的原型链上(并非每个实例本身上)，如果该属性是一个引用类型，这会造成一个实例的修改也会影响到其他的实例。

``` javascript
function Foo() {
  this.ref = {name: 'foo'};
}
function Bar() {}
Bar.prototype = new Foo();

var bar1 = new Bar();
console.log(bar1.ref.name);
// 'foo'

var bar2 = new Bar();
console.log(bar2.ref.name);
// 'foo'

bar2.ref.name = 'bar2'; 
console.log(bar1.ref.name);
// 'bar2'
```

③ 通该子类`Car`构造的实例的`constructor`指向父类`Vehicle`。这是因为对象的`constructor `属性其实是指向其 *[[Prototype]]* 的`constructor`，当然只需复写一下该属性即可。

``` javascript
console.log(car.constructor);
// Vehicle

Car.prototype.constructor = Car;
console.log(car.constructor);
// Car
```

#### 借用构造函数继承
``` javascript
function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}

var car = new Car(2, 4);
console.log(car.engines);
// 2
console.log(car.wheels);
// 4
```

- 通过父类构造函数借用的方式，可以实现实例化子类对象时向父类构造函数动态传参(即上面提到的问题①)。
- 而且绑定`this`子类实例对象借用父类构造函数，通过父类构造函数初始化的属性会直接绑定到实例化的每一个对象上，因此不会存在多个实例之间互相影响的问题(即上面提到的问题②)。

但是单独使用该方式很少，因为不涉及到原型链继承没有太大意义。

#### 混合继承
即上面的原型链继承与构造函数借用继承相结合，弥补各自的不足。

``` javascript
function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}

Car.prototype = new Vehicle();
Car.prototype.constructor = Car;

Car.prototype.drive = function() {
  console.log('Steering ' + this.wheels + ' wheels and moving forward!');
}

var car = new Car(3, 4);
console.log(car.engines);
// 3
console.log(car.hasOwnProperty('engines'));
// true
car.ignition();
// Turning on 3 engines!
```
通过该方式继承仍有两个缺点

① 父类`Vehicle`的构造方法调用了两次，造成了不必要的内存消耗。

② 在上例中，当将子类的`prototype`指向父类的`prototype`时调用并没有传任何参数，这是因为我们这时只想关联`prototype`，并不关心他到底需要什么参数。在上例中并不会有什么大问题，但是这种操作存在隐患，有时会给我们带来意想不到的副作用。

``` javascript
function Foo(arr, ...rest) {
  this.arr = [...arr, ...rest];
}
Foo.prototype.print = function() {
  console.log(this.arr);
}

function Bar() {}

Bar.prototype = new Foo();
// Uncaught TypeError: ref is not iterable
```
当然这是我们假象的一个例子，本身是由于代码不够健壮造成的。你可以改造Foo的构造函数，检查参数或者配置默认参数。这里想表达的是，其实我们这里根本不需要执行`Foo`构造函数，因为我们只关心其`prototype`

#### 直接指向父类prototype
该方法没有任何使用价值，只是为了说明问题。

``` javascript
function Car() {}

Car.prototype = Vehicle.prototype;
Car.prototype.constructor = Car;

var vehicle = new Vehicle(1);
consle.log(vehicle.constructor);
// Car
```
该方法直接将子类的`prototype`指向父类的`prototype`，避免了实例化父类对象，节省了内存。但是子类对`prototype`的任何修改都会直接影响到父类以及其他子类，因为`prototype`是同一个对象引用。

#### 利用空对象继承
该方法是在上面的基础上演化而来的。

``` javascript
function F() {}
F.prototype = Vehicle.prototype;

function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}

Car.prototype = new F();
Car.prototype.constructor = Car;
```

由于直接将子类的`prototype`指向父类的`prototype`会存在上述问题，而且前面提到多次调用父类构造方法以及空着调用父类构造函数可能会导致副作用。该方法利用一个空对象作为介质，实例化`F`几乎不占内存，而且修改子类的`prototype`也不会影响到父类的`prototype`。

通过同样的思路(指向子类的`prototype`到一个空对象)，该方法还可以通过下面一种实现：

``` javascript
function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}

Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;
```
`Object.create(o: Object)`是ES5中的一个方法，接收一个对象`o`，返回一个空对象，该返回对象的`prototype`会指向传入的参数`o`。所以该实现和上面的构造临时`F`是等价的，而且由于该方法省去了空类`F`的创建，所以会更加推荐使用此方法。

#### 拷贝继承
以上的方法都是通过子类关联父类的原型链的方式继承。其实我们也可以直接从父类的原型链上面将这些不变的属性/方法拷贝到子类的原型链上面，这样不也能实现子类调用父类的方法了吗？

``` javascript
function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}

var vp = Vehicle.prototype;
var cp = Car.prototype;

for (var i in vp) {
  cp[i] = vp[i];
}
```
这里需要注意的是，由于`for in`还会遍历继承下来的属性，但是并不会遍历`enumerable`为`false`的属性。

``` javascript
function Foo() {}
Foo.prototype.foo = function() {
  console.log('foo');
}

function Bar() {}
Bar.prototype = new Foo();
Bar.prototype.bar = function() {
  console.log('bar');
}
Object.defineProperty(Bar.prototype, 'bla', {
  enumerable: false,
  value: () => {
    console.log('bla');
  }
})

function Baz() {}
for(var i in Bar.prototype) {
  Baz.prototype[i] = Bar.prototype[i];
}

var baz = new Baz();

baz.bar();
// bar
baz.foo();
// baz
baz.bla;
// undefined
```

## 并不存在的 类
当在真正弄清楚在Javascript中原型链`prototype`的工作方式之后，你会发现这和传统的面向对象比如Java存在很大区别的。

在Javascript中，所谓的*类的继承*无非是实例通过 *[[Prototype]]* 一层一层向上查找实现的。与其说是*继承*，不如说是**委托/代理**，子类实例想要使用父类定义的方法，不就是将该方法委托给父类的`prototype`吗？一旦父类在`prototype`中定义的方法发生改变，子类再调用该方法就是改变后的方法，因为委托的对象发生了改变。

### “类的多态”
在面向对象编程里面，多态是最重要的概念之一。多态的定义是接口的多种不同实现，在调用时父类类型的引用指向子类对象。

由于Javascript中并不存在类，更不存在接口。然而人们已经接受了面向对象的思维去理解原型链，那么来看下是怎么模拟多态的。

``` javascript
function Car(engines, wheels) {
  Vehicle.call(this, engines);
  this.wheels = wheels;
}
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.ignition = function() {
  Vehicle.prototype.ignition.call(this);
  console.log('Rolling on ' + this.wheels + ' wheels.');
}

function SpeedBoad(engines) {
  Vehicle.call(this, engines);
}
SpeedBoad.prototype = Object.create(Vehicle.prototype);
SpeedBoad.prototype.ignition = function() {
  Vehicle.prototype.ignition.call(this);
  console.log('Speeding through the water.');
}

var car = new Car(2, 4);
car.ignition();
// Turning on 2 engines!
// Rolling on 4 wheels.

var speedBoad = new SpeedBoad(2);
speedBoad.ignition();
// Turning on 2 engines!
// Speeding through the water.
```
上面代码通过在子类的`prototype`上定义与父类同名的`ignition`方法，达到遮蔽父类方法的目的，实现了相对多态的效果。


### 行为委托
上面提到在类的世界里，通过定义一个同名的方法达到相对多态的目的。但是如果我们面对现实，不用类的思维去思考问题，在我们面前的紧紧是一个个普通的对象`object`而已。这时定义同名方法往往使得代码难以理解，我们更倾向于定义另外一个方法，在这个方法里面去委托别的方法来完成相关联的逻辑。

``` javascript
var Widget = {
  init: function(width, height) {
    this.width = width;
    this.height = height;
    this.$elem = null;
  },
  insert: function($context) {
    if (this.$elem) {
      this.$elem.css({
        width: width,
        height: height
      }).appendTo($context);
    }
  }
}

var Button = Object.create(Widget);

Button.setup = function(width, height, label) {
  this.init(width, height);
  this.label = label || 'Button';
  
  this.$elem = $('<button />').text(this.label);
}
Button.build = function($context) {
  this.insert($context);
  this.$elem.click(this.onClick.bind(this));
}
Button.onClick = function(evt) {
  console.log('Button ' + this.label + ' clicked.');
}

var $body = $(document.body);
var btn = Object.create(Button);

btn.setup(100, 20, 'Hello');
btn.build($body);
```
上面代码中，完全通过纯对象的思维去定义了基础组件`Widget`与一般组件`Button`的关系。在`Button`的`setup`方法中委托`Widget.init`方法，并添加了自己特有的逻辑，从而达到了代码抽象与重用的目的。

## class in ES6
ES6提供了`class`的新语法，不过只是一个语法糖而已，本质还是前面提到的通过原型链`prototype`的实现。下面我们来看一下使用ES6是如何定义类的。

``` javascript
class Vehicle {
  constructor(engines) {
    this.engines = engines;
  }
  
  ignition() {
    console.log('Turning on ' + this.engines + ' engines!');
  }
}

class Car extends Vehicle {
  constructor(engines, wheels) {
    super(engines);
    this.wheels = wheels;
  }
  
  ignition() {
    super.ignition();
    console.log('Rolling on ' + this.wheels + ' wheels.');
  }
}

const car = new Car(2, 4);

car.ignition();
// Turning on 2 engines!
// Rolling on 4 wheels.
```

我们可以看到通过ES6的方式去定义一个类以及继承的关系变得清爽了很多，其实它只是把`prototype`隐藏了起来。当通过`extends`继承父类时，可以通过`super.ignition()`调用父类中定义的方法，其实它等于`Vehicle.prototype.ignition.call(this)`。

### ES6 class 那些小事
1. **定义class的成员变量和静态属性**

	到目前为止，官方版本要定义class的成员变量只能通过在`constructor`中通过`this.props = props`的方式来达到。那如何定义class的静态属性呢？就上面例子，可以通过`Vehicle.version = 1`这种方式来定义。
	
	其实可以有更清晰友好的方式，但是目前没在ES6的正式版本中。如果使用babel，加入 *stage-2* 即可(stage提案的执行阶段)。然后就可以这样玩了。
	
	``` javascript
	class Vehicle {
	  timeCreated = new Date();
	  static version = '1.0';
	}
	```
	
2. **理清关系**

	``` javascript
	class A {
	}
	
	class B extends A {
	}
	
	B.__proto__ === A;
	// true
	B.prototype.__proto__ === A.prototype;
	// true
	```
	由于每个对象都有 *[Prototype]* 属性，指向构造该对象的构造函数的的`prototype`属性。class的本质是一个普通`function`，同时也是一个普通对象。当通过`extends`实现继承时，而同时存在`prototype`和 *[Prototype]* 两条继承链。
	
	(1) 子类的 *[Prototype]* 属性，表示**构造函数**的继承，总是指向父类。

	(2) 子类`prototype`属性的 *[Prototype]* 属性，表示**方法**的继承，总是指向父类的`prototype`属性。
	
3. **super问题**

	我们上面提到了当调用`super.xxx`时，其实内部是会从父类的`prototype`上查找。所以任何不存在`protoype`上的属性/方法是无法通过`super`得到的。
	
	``` javascript
	class Vehicle {
     constructor(engines) {
       this.engnes = engines;
     }
	}
	
	class Car extends Vehicle {
	  constructor(engines) {
	    super(engines);
	  }
	  
	  print() {
	    console.log('super: ' + super.engines);
	    console.log('self: ' + this.engines);
	  }
	}
	
	const car = new Car(2);
	car.print();
	// super: undefined
	// self: 2
	```
	
4. **访问权限**

	ES6并没有提供对于class的访问权限控制，因此也不存在私有变量/方法。但是如果你一定想要，也可以达到类似的效果。
	
	``` javascript
	const _log = Symbol();
	
	class Vehicle {
     constructor(engines) {
       this.engnes = engines;
     }
     
     [_log]() {
       const now = new Date();
       console.log(`[${now}]: logging`);
     }
     
     ignition() {
	    console.log('Turning on ' + this.engines + ' engines!');
	    this[_log]();
	  }
	}
	
	export default Vechicle;
	```
	上面的示例中利用ES的`Symbol`每次运行得到的都是不一样的值的特性，将需要私有的属性封装在模块内部，然后再通过模块的`export`将该class暴露出去。