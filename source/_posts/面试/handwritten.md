---
title: 面试痛点：手写常见代码
date: 2019-10-31 14:53:51
tags:
  - javascript
categories:
  - 面试
keywords: 'js,面试'
description: 面试官：来手写一个xxx，然后再手写一个xxx <br /> 我：???
cover: https://cdn.jsdelivr.net/gh/LishiJ/assets/img/handwritten.jpeg
---

> 面试官：来手写一个xxx，然后再手写一个xxx <br /> 我：???

俗话说金九银十，很多人想着趁着这个时候能跳槽到一个不错的公司，可是有的人一到面试就崩，比如我们经常遇到的面试题：手写一个xxx。

很多人在栽在这个上面了，这也是我们平常所说的会用却不知道其原理，所以说要想找到一个满意的工作还是得自身过硬啊！

也有很多人问，开发时又不用自己手写，为啥面试非得考这种问题呢？我会用不就好了？其实这是考验一个人的解决问题的能力，遇到问题能够更快更准确的定位和解决问题，这是一个公司所需要的，也是平常人与优秀的人的差距所在。下面就让我们一起看看经常会遇到的一些手写常见代码吧！

## new

我们要手写一个方法，首先要清楚这个方法做了什么事，然后我们按步骤一步步写出来就好了。

`new` 操作符做了：

1. 创建一个空对象

2. 将空对象的原型赋值为构造函数的原型，即空对象的 `__proto__` 指向构造函数的 `prototype`

3. 将构造函数内的的 `this` 指向空对象，如果 `return` 出来东西是对象的话就直接返回  `return` 的内容，没有的话就返回创建的这个对象

清楚了 `new` 操作符实现的事情以后，就比较简单了，我们通过代码一步步来实现下：

``` js
function myNew(func) {
  // step1: 创建一个空对象
  const obj = {};
  // step2: 空对象的原型赋值为构造函数的原型
  obj.__proto__ = func.prototype;
  // step3: 绑定 this，并判断返回的是否为对象
  const returnObj = func.apply(obj, Array.prototype.slice.call(arguments, 1));
  const type = typeof returnObj;
  if ((type === 'object' || type === 'function') && returnObj !== null) {
    return returnObj;
  }
  return obj;
}
```

## JSON.stringify

`JSON.stringify` 是 `json` 序列化的一个操作，这里我们需要的有几点：

* `Boolean|Number|String` 类型会自动转换成对应的原始值。

* `undefined`、任意函数以及 `symbol`，会被忽略（出现在非数组对象的属性值中时），或者被转换成 `null`（出现在数组中时）。

* 不可枚举的属性会被忽略

* 如果一个对象的属性值通过某种间接的方式指回该对象本身，即循环引用，属性也会被忽略。

``` js
function jsonStringify(obj, isArr = false) {
    let type = typeof obj; // object (function undefined symbol) (number string boolean)

    // 简单类型
    const isSimple = /boolean|number|string/.test(type);
    if (type !== "object" || obj === null) {
        if (isSimple || obj === null) {
            if (/string|function/.test(type)) {
                obj = '"' + obj + '"'
            }
            return String(obj);
        } else if (isArr) {
            return null
        } else {
            return undefined
        }
    } else {
        let json = []
        let _isArr = (obj && obj.constructor === Array);
        for (let k in obj) {
            let v = obj[k];
            let type = typeof v;
            v = jsonStringify(v, _isArr)
            if (v !== undefined) {
                json.push((_isArr ? "" : '"' + k + '":') + String(v))
            }
        }
        return (_isArr ? "[" : "{") + String(json) + (_isArr ? "]" : "}")
    }
}
```

## JSON.parse

`JSON.parse` 是与 `JSON.stringify` 相对的用于将 `json` 字符串转成 `JavaScript` 值或对象。

### evel

``` js
function jsonParse(opt) {
	return eval('(' + opt + ')')
}
jsonParse(jsonStringify({ x: 5}))
jsonParse(jsonStringify([1, "false", false]))
jsonParse(jsonStringify({b: undefined}))
```

看上面的代码非常简单就可以实现，但是我们知道应该避免在不必要的情况下使用 `eval`，`eval()` 是一个危险的函数，他执行的代码拥有着执行者的权利。如果你用 `eval()` 运行的字符串代码被恶意方（不怀好意的人）操控修改，您最终可能会在您的网页/扩展程序的权限下，在用户计算机上运行恶意代码。

它会执行 `JS` 代码，有 `XSS` 漏洞。

如果你只想记这个方法，就得对参数 `json` 做校验。

``` js
var rx_one = /^[\],:{}\s]*$/;
var rx_two = /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g;
var rx_three = /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g;
var rx_four = /(?:^|:|,)(?:\s*\[)+/g;
if (rx_one.test(json.replace(rx_two, "@").replace(rx_three, "]").replace(rx_four, ""))) {
	var obj = eval("(" + json + ")")
}
```

### Function

``` js
var jsonStr = '{ "age": 20, "name": "jack" }'
var json = (new Function('return ' + jsonStr))();
```

`eval` 与 `Function` 都有着动态编译js代码的作用，但是在实际的编程中并不推荐使用。

## call & apply

`call` 和 `apply` 是我们比较熟悉的两个语法，用于调用一个函数，唯一的区别在于参数的传递方式，`call` 分别地提供参数，`apply` 提供一个参数数组。

``` js
fun.call(thisArg, arg1, arg2, ...)
func.apply(thisArg, [argsArray])
```

name `call` 和 `apply` 核心是怎么实现的呢？简单的可以概括下面几点：

- 将函数设为对象的属性

- 执行 & 删除这个函数

- 指定 `this` 到函数并传入给定参数执行函数

- 如果不传入参数，默认指向为 `window`

``` js
Function.prototype.myCall = function (content = window) {
    // 将函数设为对象的属性
    // 指定 `this` 到函数并传入给定参数执行函数
    content.fn = this;
    // 获取参数
    let args = [...arguments].slice(1);
    // 执行函数
    let result = content.fn(...args);
    // 删除函数
    delete content.fn;
    // 返回结果
    return result;
}

const foo = {
	value: 1
}
function bar(name, age) {
    console.log(name) 
    console.log(age) 
    console.log(this.value)
}
bar.myCall(foo, 'black', '18')
```

其实 `apply` 和上面实现方法一样，只是将获取参数的方式进行改变了而已。

``` js
Function.prototype.myApply = function (content = window) {
    // 将函数设为对象的属性
    // 指定 `this` 到函数并传入给定参数执行函数
    content.fn = this;
    // 获取参数 & 执行函数
    let result = content.fn(...arguments[1]);
    // 删除函数
    delete content.fn;
    // 返回结果
    return result;
}

const foo = {
	value: 1
}
function bar(name, age) {
    console.log(name) 
    console.log(age) 
    console.log(this.value)
}
bar.myApply(foo, ['black', '18'])
```

## bind

`bind` 虽然可以像 `call` 和 `apply` 一样进行 `this` 的 绑定，但是 `bind` 会创建一个新的函数。此外，`bind` 实现还需要考虑实例化后对原型链的影响。

``` js
Function.prototype.myBind = function(content) {
    // 是否为函数_若不考虑参数类型可以省略
    if (typeof this !== 'function') {
        throw Error("not a function.")
    }
    // 获取参数
    let fn = this;
    let args = [...argumens].slice(1);

    // 创建一个新的函数
    let resFn = function () {
        // 判断是否继承于该函数
        return fn.apply(this.instanceof resFn ? this : content, args.concat(...arguments))
    }
    // 返回一个新的函数
    return resFn;
}
```

## 继承

继承方法有多重，这里就不详述了，但是我们一般使用最多同时也是缺点较少的就是**寄生组合继承**了。

核心就是：用一个 `F` 空的构造函数去取代执行了 `Parent` 这个构造函数。

``` js
// 父类
function Parent(name) {
    this.name = name;
}
Parent.prototype.sayName = function() {
    console.log("Parent name is " + this.name);
}

// 子类
function Child (name, parentName) {
    Parent.call(this, parentName);
    this.name = name;
}

// 继承方法
function create (parent) {
    function F() {};
    F.prototype = parent.prototype;
    return new F();
}

Child.prototype = create(Parent);
Child.prototype.sayName = function() {
	console.log('child name:', this.name)
}
Child.prototype.constructor = Child;
var parent = new Parent('father');
parent.sayName();
var child = new Child('son', 'father');
```

不是太理解的可以先过，建议后面抽空多理解学习多种继承方式，比较实用。

## JS函数柯里化

> 在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数且返回结果的新函数的技术。

函数柯里化的主要作用和特点就是**参数复用**、**提前返回**和**延迟执行**。

``` js
function curry() {
	var args = Array.prototype.slice.call(arguments);
	var fn = function() {
		var newArgs = args.concat(Array.prototype.slice.call(arguments));
		return multi.apply(this, newArgs)
	}
	fn.toString = function() {
		return args.reduce(function(a, b) {
			return a * b
		})
	}
	return fn
}
// ES6骚操作
const curry = (fn, arr = []) => (...args) => (
    arg => arg.length === fn.length
        ? fn(...arg)
        : curry(fn, arg)
)([...arr, ...args])


function multiFn(a, b, c) {
	return a * b * c
}
var multi = curry(multiFn);


multi(2)(3)(4);
multi(2, 3, 4);
multi(2)(3, 4);
multi(2, 3)(4);

```

## Promise

`Promise` 是比较常用的，首先我们要知道 `Promise` 有三种状态：`pending` | `fulfilled(resolved)` | `rejected`；当处于 `pending` 状态的时候，可以转移到 `fulfilled(resolved)` 或者 `rejected` 状态；当处于 `fulfilled(resolved)` 状态或者 `rejected` 状态的时候，就不可变，最后须有一个 `then` 异步执行方法，`then` 接受两个参数且必须返回一个 `promise`。

虽然这部分代码比较多，但是这边我还是基于大家都比较清楚地了解 `promise` 的情况下写的注释，所以直接上代码（如果看代码不是太清楚的话建议去更多的了解下 `promise`）：

``` js
// 定义状态
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

// myPromise
function myPromise (excutor) {
    // 保存当前 promise 对象
    let _this = this;
    // 初始化状态
    _this.status = PENDING;
    // 用于存储 fulfilled 时的值
    _this.value = undefined;
    // 用于存储 rejected 时的值
    _this.reason = undefined;
    // 用于存储 fulfilled 状态对应的 onFulfilled 函数
    _this.onFulfilledCallbacks = [];
    // 用于存储 rejected 状态对应的 onRejected 函数
    _this.onRejectedCallbacks = [];

    /**
     * resolve
     * @param {any} value
     */
    function resolve(value) {
        if (value instanceof myPromise) {
            return value.then(resolve, reject);
        }
        // 确保异步执行
        setTimeout(() => {
            if (_this.status === PENDING) {
                _this.status = FULFILLED;
                _this.value = value;
                _this.onFulfilledCallbacks.forEach(cb => cb(_this.value));
            }
        })
    }
    /**
     * reject
     * @param {any} reason
     */
    function reject(reason) {
        setTimeout(() => {
            if (_this.status === PENDING) {
                _this.status = REJECTED;
                _this.reason = reason;
                _this.onRejectedCallbacks.forEach(cb => cb(_this.reason));
            }
        })
    }

    // 捕获 excutor 执行器中的异常
    try {
        excutor(resolve, reject);
    } catch (err) {
        reject(err);
    }
}

/**
 * 解析 then 返回值与新 Promise 对象
 * @param {Object} newPromise 新的 Promise 对象 
 * @param {*} value 上一个 then 的返回值
 * @param {Function} resolve newPromise 的 resolve
 * @param {Function} reject newPromise 的 reject
 */
function resolvePromise(newPromise, value, resolve, reject) {
    // 是否为循环调用
    if (newPromise === value) {
        reject(new TypeError('Promise 发生了循环调用.'));
    }

    // value 为 Promise
    if (value !== null && (typeof value === 'object' || typeof value === 'function')) {
        // 对象或者函数
        try {
            let then = value.then;  //取出then方法引用
            if (typeof then === 'function') {
                then.call(value, _value => {
                    resolvePromise(newPromise, _value, resolve, reject);
                }, err => {
                    reject(err);
                })
            } else {
                resolve(value);
            }
        } catch (e) {
            reject(e);
        }
    } else {    // 普通值
        resolve(value);
    }
}

// 在原型上定义 then 方法
myPromise.prototype.then = function (onFulfilled, onRejected) {
    let _this = this;
    let newPromise;

    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected: reason => {
        throw reason;
    }

    // 成功态
    if (_this.status === FULFILLED) {
        return newPromise = new myPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let _value = onFulfilled(_this.value);
                    resolvePromise(newPromise, _value, resolve, reject);
                } catch (err) {
                    reject(err);
                }
            })
        })
    }

    // 失败态
    if (_this.status === REJECTED) {
        return newPromise = new myPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let _reason = onRejected(_this.reason);
                    resolvePromise(newPromise, _reason, resolve, reject);
                } catch (err) {
                    reject(err);
                }
            })
        })
    }

    // 等待态
    if (_this.status === PENDING) {
        return newPromise = new myPromise((resolve, reject) => {
            _this.onFulfilledCallbacks.push(value => {
                try {
                    let _value = onFulfilled(value);
                    resolvePromise(newPromise, _value, resolve, reject);
                } catch (err) {
                    reject(err);
                }
            })
            _this.onRejectedCallbacks.push(reason => {
                try {
                    let _reason = onRejected(reason);
                    resolvePromise(newPromise, _reason, resolve, reject);
                } catch (err) {
                    reject(err);
                }
            })
        })
    }
}
```

到这边，我们的 `promise` 就写完了，要想测试我们所写的代码是否正确，可以在[这里](https://github.com/promises-aplus/promises-tests)进行测试哦~

## 防抖(Debouncing)和节流(Throttling)

> 当一次事件发生后，事件处理器要等一定阈值的时间，如果这段时间过去后 再也没有 事件发生，就处理最后一次发生的事件。假设还差 0.01 秒就到达指定时间，这时又来了一个事件，那么之前的等待作废，需要重新再等待指定时间。

``` js
function debounce(fn, wait = 50, immediate) {
	let timer;
	return function() {
		if (immediate) {
			fn.apply(this, arguments)
		}
        if (timer) clearTimeout(timer)

        timer = setTimeout(() => {
			fn.apply(this, arguments)
		}, wait)
	}
}
```

> 可以理解为事件在一个管道中传输，加上这个节流阀以后，事件的流速就会减慢。实际上这个函数的作用就是如此，它可以将一个函数的调用频率限制在一定阈值内，例如 1s，那么 1s 内这个函数一定不会被调用两次

``` js
function throttle(fn, wait) {
	let prev = new Date();
	return function() {
		const args = arguments;
		const now = new Date();
		if (now - prev > wait) {
			fn.apply(this, args);
			prev = new Date()
		}
    }
}
```

## 浅拷贝 & 深拷贝

> 浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

### 浅拷贝

实现浅拷贝常用有两种方式（其它方式不详述）: 

- 使用 `for` `in` 循环，遍历每一个属性，将他们赋值给新的对象。 
- `Object.assign()`

#### for in

``` js
function copy (obj) {
    let _obj = {};
    for (let k in obj) {
        _obj[k] = obj[k]
    }
    return _obj
}
```

#### Object.assign()

`Object.assign()` 方法可以把任意多个的源对象自身的可枚举属性拷贝给目标对象，然后返回目标对象。但是 `Object.assign()` 进行的是浅拷贝，拷贝的是对象的属性的引用，而不是对象本身。

*注：只有一层时为深拷贝*

``` js
var obj = { a: {a: "kobe", b: 39} };
var initalObj = Object.assign({}, obj);
initalObj.a.a = "wade";
console.log(obj.a.a); //wade
```

这边稍微讲一下数组的常见的浅拷贝方法：

``` js
// Array.prototype.concat()
let arr = [1, 3, {
    username: 'kobe'
    }];
let arr2=arr.concat(); 

// Array.prototype.slice()
let arr3 = arr.slice();
```

### 深拷贝

### JSON.parse()

``` js
let arr = [1, 3, {
    username: ' kobe'
}];
let arr4 = JSON.parse(JSON.stringify(arr));
arr4[2].username = 'duncan'; 
console.log(arr, arr4)
```

### 递归

``` js
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};    
  for (var i in initalObj) {        
    var prop = initalObj[i];        // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {            
      continue;
    }        
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : {};            
      arguments.callee(prop, obj[i]);
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
var str = {};
var obj = { a: {a: "hello", b: 21} };
deepClone(obj, str);
console.log(str.a);
```

### Object.create()

``` js
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};    
  for (var i in initalObj) {        
    var prop = initalObj[i];        // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {            
      continue;
    }        
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : Object.create(prop);
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
```

## instanceOf

``` js
function instanceOf(left, right) {
	let proto = left.__proto__;
	let prototype = right.prototype
	while (true) {
		if (proto == null) return false
        if (proto == prototype) return true
        proto = proto.__proto__
	}
}
```

## 最后

看完以后是不是觉得还是比较容易理解的，所以说解决问题一定要分析问题，分析清楚了，一步步往下做就好了。

---

如果需要上面的代码及 `demo` 的话，可以点击[这里](https://github.com/LishiJ/hand_written)获取。
