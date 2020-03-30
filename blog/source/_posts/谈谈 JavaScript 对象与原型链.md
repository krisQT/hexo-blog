---
title: 谈谈 JavaScript 对象与原型链
tags: 
      - js
comments: true
categories: web前端
description: 当我们在谈继承时，其实在谈什么？
---
# 前言
写这边文章的原因：最近刚用vue写完一个项目，总感觉只用到了这个框架的形，而没有体会其中的精髓，遂想重新理一遍js。既然是谈js，那当然要从对象说起。

# 一、对象 
同其他语言类似，在 JavaScript 中，一个对象就是一系列属性的集合，一个属性包含一个名和一个值。
## 1、创建对象的方式
在 JavaScript中，有着三种基本的创建对象的方式，在这个基础上，衍生出许多另外的方式。但万变不离其宗，变的只是模式。
- 通过字面量创建
```
let obj = {
    name: "kris",
    age: 18
}
```

- 通过构造函数创建
```
let Foo = function(name, age) {
    this.name = name,
    this.age = age
} 

let obj = new Foo("kris", 18); 
```

- Object.create(proto, [propertiesObject])
```
//目的是以现在的一个对象创建一个新的对象
let shape = {
    x: 100,
    y: 100
}
let obj = Object.create(shape, {color: {value: "red"}})
```
虽然有这三种方法，但js的内部实现应该只有第二种，那就是通过 new 一个构造函数创建对象。第一种字面量方式只是 JavaScript 的语法糖，减少代码量，第三种我们到后面的原型再讲。

## 2、new 一个构造函数时，发生了什么？
还是上面的code
```
let Foo = function(name, age) {
    this.name = name,
    this.age = age
} 

Foo.prototype.description = "notiong"

Foo.prototype.sayHi = function() {
    alert('hi')
}

let obj = new Foo("kris", 18); 
```
当执行 new Foo("kris", 18)时，会发生一下事情:
- 1、一个继承 Foo.prototype 的对象被创建。
- 2、调用 Foo ，并将 **this** 绑定到新创建的对象上，在 **this** 上定义这个对象的私有属性。
- 3、构造函数返回对象。 

一个基本概念是，共享属性可以绑定到 **prototype**对象上，以达到继承的目的，让后续 new 的对象都可以访问到这些属性。需要注意到的是，对这些共享属性的更改，会影响到所有继承该 **prototype**的对象。
```
    let obj1 = new Foo("Dina", 16)
    Foo.prototype.description = "a beautiful girl"
    console.log(obj.description) //a beautiful girl
```

## 3、 prototype 和 __proto__
先上图
![avatar](http://o6rbt5px2.bkt.clouddn.com/1533264072%281%29.jpg)
<br>
当我们在谈继承时，其实本质就是在谈 prototype。
首先给出结论：
- **prototype** 只有函数对象才会有的属性, **prototype的constructor** 属性指向构造函数本身。
- **\_\_proto__** 是对象的私有属性， 指向构建它的构造函数的 prototype 。
- **prototype** 对象也会有它自己的 **\_\_proto__** , 层层往上，最终指向   **Object.prototype** ，且 Object.prototype.\_\_proto__的值 为 null。
- 以上环节构成一个原型链，且 null 作为这个环节的最后一个环节

以上部分 code 为例
```
let Foo = function(name, age) {
    this.name = name,
    this.age = age
} 

Foo.prototype.description = "notiong"

Foo.prototype.sayHi = function() {
    alert('hi')
}

let obj = new Foo("kris", 18)
```
我们可以经过测试得知：
```
obj.__proto__ == Foo.prototype //true

Foo.prototype.constructor == Foo //true

Foo.prototype.__proto__ == Object.prototype //true

Foo.__proto__ == Function.prototype //true

Object.__proto__ == Function.prototype //true 

Object.prototype.__ == null //true
```
- 可以看到， \_\_proto__ 的显式作用在于连接构造函数的 prototype 。而隐式作用是构成原型链。当我们访问对象的某个属性，如果找不到该属性就会沿着 \_\_proto__往上查找，直到找到为止。在本例中，访问 obj.description ，首先会在obj对象中查找，查找不到，就会去沿着 \_\_proto__ 去查找， 直到找到 description 或者 prototype 等于 null 为止。

- 所有函数对象的 \_\_proto__ 属性指向了 Function 的原型对象，这里函数对象包括了 function 定义的函数，也包含了 Object， Array等等构造函数。原因是所有的函数对象皆由 Function 构建而成。

- 有趣的是，Object.\_\_proto__ 等于 Function.prototy ，而 Function.prototype.\_\_proto__ 等于 Object.prototype， 理解起来是不是有点绕。


# 二、继承

## 1、原型链继承
```
// 第一层动物
function Animal(){ }
Animal.prototype.species = "动物"

// 第二层类别
function Cat(name, color){ 
	this.name = name
	this.color = color
}
var F = function(){} //空对象作为介质，避免 Cat 的 prototype 直接等于 Animal 的 prototype
F.prototype = Animal.prototype
Cat.prototype = new F()
Cat.prototype.constructor = Cat
Cat.prototype.family = "猫"

//第三层
function Ju (weight) {
	this.weight = weight
}
var Fo = function () {} //原理同上
Fo.prototype = Cat.prototype
Ju.prototype = new Fo()
Ju.prototype.constructor = Ju
Ju.prototype.name = "大橘"
var daju = new Ju(20)

console.dir(daju)
```
![avatar](http://o6rbt5px2.bkt.clouddn.com/1533283592%281%29.jpg)
<br>
以上代码就是 JavaScript 关于原型链继承的原理。
- 1>. 如果构造函数A的 prototype 属性是另一个构造函数B的实例对象，那么构造函数A会继承构造函数B的 prototype，A的实例对象可以通过 \_\_proto__ 访问到B的 prototy 对象，以达到继承的目的。

- 2>. 在上面例子中，两个空对象介质的作用在于避免构造函数直接继承 prototype ，造成原型链污染。

- 3>. 当我们将 Cat.prototype = new F() , Cat.prototype.constructor 也将会没有，规范上， constructor应该指向构造函数本身，所以应该重新定义 Cat.prototype.constructor = Cat。不过这个 constructor 实际作用不大。

现在我们再来理解上述的第三种构建对象的方式。在MDN上这样定义这个函数：Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。
```
const person = {
  isHuman: false,
  printIntroduction: function () {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`)
  }
};

const me = Object.create(person)

me.name = "Matthew" // "name" is a property set on "me", but not on "person"
me.isHuman = true // inherited properties can be overwritten

me.printIntroduction()
// expected output: "My name is Matthew. Am I human? true"
```
通过之前的知识点，可以理解到其中的写法应该是这样的 
```
function create(o){
    function F(){}
    F.prototype = o;
    return new F()
}
```
应用的也就是原型链继承的原理。

## 2. es6 的 class 与继承。
JavaScript 的原型链继承是整门语言的精髓所在， 但也是最难理解的部分。为了更容易使用， es6 使用了 class 这个语法糖，但原理任然是基于原型链的。

### 1. class
用法
```
class Animal {
    constructor(name) {
        this.name = name
    }

    sayHi () {
        console.log('hi')
    }
}

//等同于下面的es5写法

function Animal(name) {
    this.name = name
}
Animal.prototype.sayHi = function () {
    console.log('hi')
}
```
这样看起来，es6似乎是更加简洁了。但其实在浏览器上控制台上查看 Animal.prototype 两者并无差别。

### 2. class的继承
```
class Cat extends Animal {
    constructor(name) {
        super(name)
    }
	miao () {
		console.log("miaomiao")
	}
}
```
class 的继承也是基于原型链的原理，所以理论上并无差别，只是在外面套了一层糖衣，使用起来没有那么麻烦。

