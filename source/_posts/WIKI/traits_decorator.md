---
title: Traits-decorator使用说明
date: 2019-8-6 15:14:44
tags:
  - javascript
  - ES6
  - traits-decorator
categories:
  - WIKI
keywords: 'ES6,traits-decorator,javascript'
description: 本文主要为基于Decorator的Mixin的实现，学习或使用过类（class）的同学们可能会遇到过这样一个问题：对象的继承问题，在一个对象中继承其他若干个对象的若干方法。
cover: https://cdn.jsdelivr.net/gh/LishiJ/assets/img/es6.jpg
---

本文主要为基于 `Decorator` 的 `Mixin` 的实现，学习或使用过类（`class`）的同学们可能会遇到过这样一个问题：对象的继承问题，在一个对象中继承其他若干个对象的若干方法。`traits-decorator` 就是来解决这个问题的，一起去看看吧！

#### 安装
---
>  使用npm
``` sh
 npm i -S traits-decorator
```

> 使用git存储库
``` sh
npm i -S git://github.com/cocktailjs/traits-decorator
```

#### API
---
`traits-decorator` 的API主要包含`Decorator` 和 `Bindings` 两部分

##### Decorator
###### @traits(Trait1, ...TraitN)
用于 `class` 的定义，它将 `traits` 的所有方法应用到当前 `class`

```js
@traits(TExample) class MyClass {}
```

###### @requires(description1, ...descriptionN)
用于 `class` 某个方法的定义，该装饰器不会做任何事情，它只是提供一个文档说明来反映该方法所需 `method` / `property`

```js
class TFoo {

    @requires('bar: {String}')
    fooBar() {
        console.log('foo,' + this.bar);
    }
}
```

##### Bindings

###### excludes(Method1, ...MethodN)
用于使用 `@traits` 处，它可以从 `traits` 中排除指定方法

```js
@traits(TExample::excludes('foo', 'bar')) 
class MyClass {}
```

###### alias(aliases: {})
用于使用 `@traits` 处， 它可以为 `traits` 中的方法起一个别名

```js
@traits(TExample::alias({baz: 'parentBaz'}))
class MyClass {}
```

###### as({alias: {}, excludes: []})
用于使用 `@traits` 处，它将从特征中应用和排除的某些方法

```js
@traits( TExample::as({alias: {baz: 'parentBaz'}, excludes:['foo', 'bar'] }) )
class MyClass {}
```


#### 使用

通过下面简单的例子我们可以很快而且清晰的知道上面的API的具体使用方法及效果

``` js
'use strict';

import { traits, excludes, alias, requires }  from 'traits-decorator'

class TFirst {

    @requires('collection:[]')
    first() {
        return this.collection[0];
    }

}

class TLast {
    
    @requires('collection:[]')
    last() {
        let collection = this.collection;
        let l = collection.length;
        return collection[l-1];
    }

    justAnother() {}

    foo() {
        console.log('from TLast\'s foo');
    }
}

//composing a Trait with others
@traits( TFirst, TLast::excludes('foo', 'justAnother') )
class TEnum {

    foo() {
        console.log('enum foo')
    }
}

//apply trait TEnum
@traits(TEnum::alias({ foo: 'enumFoo' }) )
class MyClass {

    constructor (collection = []) {
        this.collection = collection
    }
}

let obj = new MyClass([1,2,3])

console.log(obj.first()) // 1

obj.enumFoo() // enum foo

```

#### github地址
[点击进入](https://github.com/CocktailJS/traits-decorator)

#### 其他
xxx