---
title: Proxy-Vue3.x数据代理
date: 2019-08-02 14:59:00
tags:
  - Vue
  - Proxy
  - ES6
categories:
  - 技术随笔
keywords: 'Vue,Vue3.0,javascript,Proxy'
description: 在之前的 Vue2.x 版本中，由于 Object.defineProperty 的限制，所以 我们在使用中遇到了无法监听属性的添加和删除、数组索引和长度的变更等一系列问题，在最新的 Vue3.0中采用 Proxy 属性很好的解决了这一问题。而且支持 Map、Set、WeakMap 和 WeakSet！下面就让我们一起去看看吧~
cover: https://cdn.jsdelivr.net/gh/seiwhale/assets/img/vue3.0.jpg
---

去年尤大大在 `VueConf TO 2018`  大会上发表了名为 `Vue3.0 Updates` 的主题演讲，对 `Vue3.0` 的更新计划、方向进行了详细阐述，表示已经放弃使用了 `Object.defineProperty`，而选择了使用更快的原生 `Proxy`。

那么 `Proxy` 有什么好处呢？

在之前的 `Vue2.x` 版本中，由于 `Object.defineProperty` 的限制，所以 我们在使用中遇到了无法监听 **属性的添加和删除**、**数组索引和长度的变更**等一系列问题，在最新的  `Proxy` 属性中很好的解决了这一问题。而且支持 `Map`、`Set`、`WeakMap` 和 `WeakSet`！下面就让我们一起去看看吧~

## 概述
首先我们需要了解一下什么是 `Proxy`， `MDN`  上是这么说的：

>  `Proxy` 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。

是不是非常的简单？对~简单到根本不知道在说什么。。。

其实可以这么简单地来理解，`Proxy` 就是在操作目标对象之前进行了"**代理**（或者说**劫持**）"，可以对外界的操作进行过滤和改写，修改操作的默认行为。这样的话我们就可以不操作对象本身，而通过**代理对象**间接地操作对象！下面我们通过一个简单的小例子来熟悉一下我们这段话的意思：

``` js
let obj = {
    name: "obj_name"
}

// new 一个 proxy 对象
let proxy_obj = new Proxy(obj, {
    get: (target, prop) => {
        console.log("proxy value get");
        return target[prop] || `${prop}未定`
    },
    set: (target, prop, value) => {
        console.log("proxy value set");
        target[prop] = value || 'proxy_name';
    }
})

console.log(proxy_obj.name);
console.log(proxy_obj.sex);

proxy_obj.name = '张三';
console.log(proxy_obj.name);
proxy_obj.name = null;
console.log(proxy_obj.name);
```

在上面的例子中会输出如下结果：

``` javascript
proxy value get
obj_name
proxy value get
sex未定
proxy value set
proxy value get
张三
proxy value set
proxy value get
proxy_name
```

## 基本使用

从上面的例子中我们能比较简单的出来 `Proxy` 是一个构造函数，使用 `new Proxy` 创建代理器，它接受两个参数，第一个参数 `target` 为被代理对象， `handler`是一个对象，其中声明了一些代理 `target` 的一些操作。

```  js
var p = new Proxy(target, handler);
 ```

从刚开始的例子中我们可以看到 `handler` 对象中我们定义了一个 `get` 和 `set` 方法，在被代理对象在赋值时触发 `set` 方法，取值时触发 `get` 方法，后面我们再详细述说有哪些操作可供我们使用~

## handler中的API

`Proxy` 的 `handler` 中目前提供了13中可代理的操作，如下所示。

``` js
handler.getPrototypeOf()
handler.setPrototypeOf()
handler.isExtensible()
handler.preventExtensions()
handler.getOwnPropertyDescriptor()
handler.defineProperty()
handler.has()
handler.get()
handler.set()
handler.deleteProperty()
handler.ownKeys()
handler.apply()
handler.construct()
```

下面我们就常用的 `API` 做一些简单的说明，有兴趣的同学可以去 `MDN` 去查阅其它的一些。

### handler.get(target, property, receiver)

`get` 用于对代理对象的属性读取操作，`target` 是指目标对象，`property` 是被获取的属性名 ， `receiver` 是 `Proxy` 或者继承 `Proxy` 的对象，一般情况下就是 `Proxy` 实例。

``` js
let proxy_obj = new Proxy({ name: '张三' }, {
    get: function (target, prop) {
        console.log(`${prop}被读取`);         
        return target[prop];
    }
})

// name被读取
// 张三
console.log(proxy_obj.name)  
```

上面的例子会很明显的知道 `target` 和 `prop` 的意义，我们通过下面这个例子来简单说明一下第三个参数 `receiver` 是不是 `Proxy` 实例呢

``` js
let proxy_obj = new Proxy({}, {
    get: function (target, prop, receiver) {       
        return receiver;
    }
})

console.log(proxy_obj.name === proxy_obj)  // true
```
上述 `proxy_obj` 对象的 `name` 属性是由 `proxy_obj` 对象提供的，所以 `receiver` 指向 `proxy_obj` 对象，因此 `proxy.a === proxy` 返回的是 `true`。

`get` 方法在使用时比较简单，但是有一点需要**注意**的是：***如果要访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同，也就是不能对其进行修改，否则会抛出异常~***

``` js
let obj = {};
Object.defineProperty(obj, "age", {
    configurable: false,
    enumerable: false,
    value: 10,
    writable: false
});

let proxy = new Proxy(obj,{
    get : function (target,prop) {
        return 20;
    }
})

console.log(proxy.age)    // Uncaught TypeError
```

上面 `age` 属性我们配置为不可写、不可配置，且值为 `10`，所以在 `get` 时返回 `20` 与属性值不等，故抛出错误，当我们将 `return 20` 修改为 `return 10` 时则可正常运行~

### handler.set(target, property, value, receiver)

通过上面我们对 `get` 方法的例子，我们可以很快的知道 `set` 方法如何使用，相比 `get` 只是多了一个 `value`值。

需要注意的一点是：***在严格模式下，set方法需要返回一个布尔值，返回 true 代表此次设置属性成功了，如果返回false且设置属性操作失败，并且会抛出一个TypeError。***

``` js
let proxy_obj = new Proxy({},{
    set: function (target, prop, value) {
        target[prop] = value;
    }
})

proxy_obj.age = 10;   // 成功赋值
```

那我们如何对 `set` 值做一些限制呢？其实我们只需要哎 `set` 方法中做一些简单的判断即可：

``` js
let proxy_obj = new Proxy({},{
    set: function (target, prop, value) {
        if(prop === 'age'){
            if ( typeof value === 'number') {
                console.log('success')
                target[prop] = value;
            } else {
                throw new Error('The variable is not an integer')
            }
        }
    }
})

proxy_obj.age = '10';   // The variable is not an integer
proxy_obj.age = 10;     // success
```

在 `get` 方法中，如果访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同。同样，在 `set` 方法中，如果目标属性是不可写及不可配置的，则不能改变它的值，即赋值无效：

``` js
let obj = {};
Object.defineProperty(obj, "age", {
    configurable: false,
    enumerable: false,
    value: 10,
    writable: false
});

let proxy = new Proxy(obj,{
    set: function (target, prop, value) {
        target[prop] = 20;
    }
})

proxy.age= 20 ;
console.log(proxy.count)   // 10
```

### handler.apply(target, thisArg, argumentsList)

用于拦截函数的调用，共有三个参数，分别是目标对象（函数）`target`，被调用时的上下文对象 `thisArg` 以及被调用时的参数数组 `argumentsList`，该方法可以返回任何值。

``` js
function sum(a, b) {
    return a + b;
}

const handler = {
    apply: function(target, thisArg, argumentsList) {
        console.log(`Calculate sum: ${argumentsList}`); 
        return target(argumentsList[0], argumentsList[1]) * 2;
    }
};

let proxy = new Proxy(sum, handler);

console.log(sum(1, 2));     // 3
console.log(proxy(1, 2));   // Calculate sum：1,2
                            // 6
```

实际上，`apply` 还会拦截目标对象的 `Function.prototype.apply()` 和 `Function.prototype.call()`，以及 `Reflect.apply()` 操作，如下：

``` js
console.log(proxy.call(null, 3, 4));    // Calculate sum：3,4
                                        // 14

console.log(Reflect.apply(proxy, null, [5, 6]));    // Calculate sum: 5,6
                                                    // 22
```

### handler.construct(target, argumentsList, newTarget)

对 `js` 基础比较熟悉的同学看到 `construct` 应该知道这个方法主要用于拦截 `new` 操作符。为了使 `new` 操作符在生成的 `Proxy` 对象上生效，用于初始化代理的目标对象自身必须具有 `[[Construct]]` 内部方法；它接收三个参数，目标对象 `target` ，构造函数参数列表 `argumentsList` 以及最初实例对象时，`new` 命令作用的构造函数。

``` js
let p = new Proxy(function() {}, {
    construct: function(target, argumentsList, newTarget) {
        console.log(newTarget === p );                          // true
        console.log('called: ' + argumentsList.join(', '));     // called：1,2
        return { value: ( argumentsList[0] + argumentsList[1] ) * 10 };
    }
});

console.log(new p(1, 2).value);      // 30
```

另外，该方法必须返回一个对象，否则会抛出异常！

``` js
var p = new Proxy(function() {}, {
    construct: function(target, argumentsList, newTarget) {
        return 2
    }
});

console.log(new p(1, 2));    // Uncaught TypeError
```

### handler.has(target, prop)

这个方法也比较简单，用于判断该对象中是否含有某个属性，可以看做是 `in` 操作的钩子。该方法接受目标对象和是否存在的属性，并最后返回 `boolean` 值。

``` js
let p = new Proxy({}, {
    has: function(target, prop) {
        if( prop[0] === '_' ) {
            console.log('it is a private property')
            return false;
        }
        return true;
    }
});

console.log('a' in p);      // true
console.log('_a' in p )     // it is a private property
                            // false
```

其余一些 `API` 有兴趣的小伙伴可以自行去查阅~

## 使用Demo

通过上面的一些了解，我们需要通过 `proxy` 的这些方法实现一些我们平常开发中可能遇到的一些问题（包括 `Vue` 中数据的绑定问题），下面就让我们一起去看看吧~

### 实现虚拟属性

实现虚拟属性即在我们的代理对象中无此属性，我们可以通过 `proxy` 来简单的实现我们想要的效果：

``` js
var person = {
  fisrsName: '张',
  lastName: '小白'
};
var proxyedPerson = new Proxy(person, {
  get: function (target, key) {
    if(key === 'fullName'){
      return [target.fisrsName, target.lastName].join(' ');
    }
    return target[key];
  },
  set: function (target, key, value) {
    if(key === 'fullName'){
      var fullNameInfo = value.split(' ');
      target.fisrsName = fullNameInfo[0];
      target.lastName = fullNameInfo[1];
    } else {
      target[key] = value;
    }
  }
});

console.log('姓:%s, 名:%s, 全名: %s', proxyedPerson.fisrsName, proxyedPerson.lastName, proxyedPerson.fullName);// 姓:张, 名:小白, 全名: 张 小白
proxyedPerson.fullName = '李 小露';
console.log('姓:%s, 名:%s, 全名: %s', proxyedPerson.fisrsName, proxyedPerson.lastName, proxyedPerson.fullName);// 姓:李, 名:小露, 全名: 李 小露
console.log('**********');
```

### 实现私有变量

我们默认以 `_` 开头的为私有变量

``` js
var api = {
  _secret: 'xxxx',
  _otherSec: 'bbb',
  ver: 'v0.0.1'
};

api = new Proxy(api, {
  get: function(target, key) {
    // 以 _ 下划线开头的都认为是 私有的
    if (key.startsWith('_')) {
      console.log('私有变量不能被访问');
      return false;
    }
    return target[key];
  },
  set: function(target, key, value) {
    if (key.startsWith('_')) {
      console.log('私有变量不能被修改');
      return false;
    }
    target[key] = value;
  },
  has: function(target, key) {
    return key.startsWith('_') ? false : (key in target);
  }
});

api._secret; // 私有变量不能被访问
console.log(api.ver); // v0.0.1
api._otherSec = 3; // 私有变量不能被修改
console.log('_secret' in api); // true
console.log('ver' in api); // false
```

### 抽离校验模块

我们可以将在 `set` 中将一些校验逻辑提取出来，通过校验函数返回结果进行赋值：

``` js
function Animal() {
  return createValidator(this, animalValidator);
}
var animalValidator = {
  name: function(name) {
    // 动物的名字必须是字符串类型的
    return typeof name === 'string';
  }
};

function createValidator(target, validator) {
  return new Proxy(target, {
    set: function(target, key, value) {
      if (validator[key]) {
        // 符合验证条件
        if (validator[key](value)) {
          target[key] = value;
        } else {
          throw Error(`Cannot set ${key} to ${value}. Invalid.`);
        }
      } else {
        target[key] = value
      }
    }
  });
}

var dog = new Animal();
dog.name = 'dog';
console.log(dog.name);
dog.name = 123; // Uncaught Error: Cannot set name to 123. Invalid.
```

### 实现数据绑定

终于来到了我们的重头戏，也就是在 `Vue` 中的双向数据绑定，下面我们来实现一个简单的双向数据绑定：

首先页面结构还是和 `2.0` 一样的，只是处理逻辑进行了变化：

``` html
<!--html-->
<div id="app">
    <h3 id="paragraph"></h3>
    <input type="text" id="input"/>
</div>
```

``` js
//获取段落的节点
const paragraph = document.getElementById('paragraph');
//获取输入框节点
const input = document.getElementById('input');

//需要代理的数据对象
const data = {
    text: 'hello world'
}

// 处理函数
const handler = {
    //监控 data 中的 text 属性变化
    set: function (target, prop, value) {
        if ( prop === 'text' ) {
                //更新值
                target[prop] = value;
                //更新视图
                paragraph.innerHTML = value;
                input.value = value;
                return true;
        } else {
            return false;
        }
    }
}

//构造 proxy 对象
const myText = new Proxy(data, handler);

//添加input监听事件
input.addEventListener('input', function (e) {
    myText.text = e.target.value;   //更新 myText 的值
}, false)

//初始化值
myText.text = data.text;   
```

通过上面的代码我们就简单的实现了一个数据的双向绑定~

## 总结

总而言之，`Proxy` 使用起来还是比较简单的，但是要想用它来实现高价值还是需要下一番功夫的。

可以等 `vue3.0` 出来后看看源码实现~

基础比较扎实的同学看过一遍后应该就比较清晰了。可能有的同学看完了还是不知所云，其实在前端这个快速发展的领域，要想紧跟潮流的脚步，最重要的还是打好基础，不管技术再怎么更新，最终都是基于基础实现的，如果仅仅只是学习新技术，那永远学不完。

最后希望各位事业蒸蒸日上~




