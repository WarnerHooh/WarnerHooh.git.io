title: ECMAScript-2017-Preview
date: 2016-12-13 20:51:24
tags: javascript
---

#ECMAScript 2017 Preview

## Array.prototype.includes
``` javascript
Array.prototype.includes（value[, index]）： boolean

> [1, 2, 3].includes(2);
// true
> [1, 2, 3].includes(2, 2);
// false
```

Why `includes`, we have `indexOf` already?

``` javascript
> [NaN].indexOf(NaN);
// -1
> [NaN].includes(NaN);
// true
```

## Exponentiation operator
``` javascript 
> let squared = 3 ** 2;
> Math.pow(3, 2);
// 9
> let num = 3;
> num ** = 2;
> console.log(num);
// 9
```

## Trailing commas in function parameter lists and calls
``` javascript
> function foo(
    param1,
    param2,
  ) {}
  
> foo(
    "a", 
    "b",
  );
  
> console.log(foo.length);
// 2
```

## String.prototype.padStart & String.prototype.padEnd
String.prototype.padStart(maxLength, fillString=' ')

``` javascript
> 'x'.padStart(5, 'ab');
// 'ababx'

> 'x'.padStart(4, 'ab');
// 'abax'

> 'abcd'.padStart(2, '#');
// 'abcd'

> 'abc'.padStart(10, '0123456789');
// '0123456abc'

> 'x'.padStart(3);
// '  x'
```

## Object.values & Object.entries
``` javascript
> let obj = {foo: "bar", baz: 7}
> Object.keys(obj);
// ["foo", "baz"]
> Object.values(obj);
// ["bar", 7]
> Object.entries(obj);
// [["foo", "bar"], ["baz", 7]]

Map map = new Map(Object.entries(obj));
// Map {"foo" => "bar", "baz" => 7}

> Object.entries("foo");
// [["0", "f"], ["1", "o"], ["2", "o"]]
```
## Object.getOwnPropertyDescriptors
``` javascript
> let obj= {};
> Object.defineProperty(obj, "foo", {
    // value: "bar",
    // writable: false,
    enumerable: true,
    configurable: false,
    get: function() {
    	return this.value;
    }
  });
  
> console.log(Object.getOwnPropertyDescriptors(obj));
// { foo:
//    { get: [Function],
//      set: undefined,
//      enumerable: true,
//      configurable: false } }
```

### Use cases
**copying properties into an object**

Since ES6, JavaScript already has a tool method for copying properties: Object.assign(). However, this method uses simple get and set operations to copy a property whose key is key. That means that it doesn’t properly copy properties with non-default attributes (getters, setters, non-writable properties, etc.).

``` javascript
> const source = {
    set foo(value) {
      console.log(value);
    }
  };
> console.log(Object.getOwnPropertyDescriptor(source, 'foo'));
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }

> const target1 = {};
> Object.assign(target1, source);
> console.log(Object.getOwnPropertyDescriptor(target1, 'foo'));
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }

> const target2 = {};
> Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
> console.log(Object.getOwnPropertyDescriptor(target2, 'foo'));
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }
```

**cloning objects**

``` javascript
> const clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
```

## Async functions
### Overview
#### Variants
``` javascript
> async function foo() {}
> const foo = async function() {}
> const foo = async () => {}
```

#### Always return Promises
``` javascript 
> async function foo() {
	return 'foo';
  }

> foo().then(x => console.log(x));
// foo

> async function bar() {
    throw new Error('Problem!');
  }
  
> bar().catch(err => console.log(err));
// Error: Problem!
```

#### Handling results and errors of asynchronous computations via *await*

``` javascript
> async function foo() {
	 try {
		const r1 = await otherAsyncFunc1();
    	console.log(r1);
   	 	const r2 = await otherAsyncFunc2();
		console.log(r2);
	 } catch(err) {
	 	console.log(err);
	 }
  }
```

### Understanding async functions
``` javascript 
// Promise
> function fetchJson(url) {
    return fetch(url)
    .then(request => request.text())
    .then(text => {
      return JSON.parse(text);
    })
    .catch(error => {
      console.log(`ERROR: ${error.stack}`);
    });
  }
> fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));

// Generator with co
> const fetchJson = co.wrap(function* (url) {
    try {
      let request = yield fetch(url);
      let text = yield request.text();
      return JSON.parse(text);
    }
    catch (error) {
      console.log(`ERROR: ${error.stack}`);
    }
  });
  
// Async 
> async function fetchJson(url) {
    try {
      let request = await fetch(url);
      let text = await request.text();
      return JSON.parse(text);
    }
    catch (error) {
      console.log(`ERROR: ${error.stack}`);
    }
  }
```

#### Async functions are started synchronously, settled asynchronously
``` javascript
> async function asyncFunc() {
    console.log('asyncFunc()');
    return 'abc';
  }
> asyncFunc().then(x => console.log(`Resolved: ${x}`));
> console.log('main');

// asyncFunc()
// main
// Resolved: abc
```

### Tips for Async functions
#### Don’t forget *await*
``` javascript
> async function asyncFunc() {
    const value = otherAsyncFunc(); // missing `await`!
  }
```
#### You don’t need await if you “fire and forget”
``` javascript
> async function asyncFunc() {
    const writer = openFile('someFile.txt');
    writer.write('hello'); // don’t wait
    writer.write('world'); // don’t wait
    await writer.close(); // wait for file to close
  }
```

#### *await* is sequential, *Promise.all()* is parallel
``` javascript
> async function foo() {
    const [result1, result2] = await Promise.all([
      asyncFunc1(),
      asyncFunc2()
    ]);
  }
```

### Async functions and callbacks
Array.prototype.map

``` javascript
> async function downloadContent(urls) {
    return urls.map(url => {
      // Wrong syntax!
      const content = await httpGet(url);
      return content;
    });
  }

> async function downloadContent(urls) {
    const promiseArray = urls.map(url => httpGet(url));
    return Promise.all(promiseArray);
  }
```
Array.prototype.forEach

``` javascript
> async function logContent(urls) {
    urls.forEach(async url => {
      const content = await httpGet(url);
      console.log(content);
    });
    // Not finished here
}

> async function logContent(urls) {
    for (let url of urls) {
      const content = await httpGet(url);
      console.log(content);
    }
}
```