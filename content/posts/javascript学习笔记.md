---
title: "Javascript学习笔记"
date: 2022-07-01T09:14:16+08:00
draft: true
---

## 面向对象

对象是引用类型和其他四种基本类型不同。JS中，可以将对象分为“**内部对象**”、“**宿主对象**”和“**自定义对象**”三种。

**内部对象**
js中的内部对象包括Array、Boolean、Date、Function、Global、Math、Number、Object、RegExp、String。
以及各种错误类对象，包括Error、EvalError、RangeError、ReferenceError、SyntaxError、TypeError。

其中Global和Math这两个对象又被称为“内置对象”，这两个对象在脚本程序初始化时被创建，不必实例化这两个对象。

**宿主对象**
宿主对象就是执行JS脚本的环境提供的对象。对于嵌入到网页中的JS来说，其宿主对象就是浏览器提供的对象，所以又称为浏览器对象，如IE、Firefox等浏览器提供的对象。不同的浏览器提供的宿主对象可能不同，即使提供的对象相同，其实现方式也大相径庭！这会带来浏览器兼容问题，增加开发难度。浏览器对象有很多，如Window和Document等等。

**自定义对象**

顾名思义，就是开发人员自己定义的对象。JS允许使用自定义对象，使JS应用及功能得到扩充

### 对象字面量

使用大括号直接创建`object`类型的对象

```javascript
var obj = {
	name:"xxx",
	age:12
}
```

### 函数对象

函数也是一个对象，使用typeof检查一个函数对象返回的是function。创建函数对象的三个方法：

使用构造函数。返回的结果是一个包含代码的对象类型

```javascript
var fun1 = new Function("console.log('Hello World')");
func1();
```

声明函数，并调用

``` javascript
function fun2(){
    console.log('Hello World');
}
func2();
```

使用参数接受一个匿名函数。匿名函数可以进行输出，内容是函数整段代码

``` javascript
var fun3 = function(){
    console.log('Hello World');
}
fun3();
```

### 函数参数

多余参数不会被赋值。

如果实参的数量少于形参的数量，则没有对应实参的形参是undefined。

### 立即执行函数

函数定义以后，立即执行，只会执行一次

``` javascript
(function(a,b){
    console.log(a+b);
})(1,2);
```

### 全局作用域

直接编写在**script**标签中的JS代码都在全局作用域，在页面打开时创建，关闭页面时销毁，全局作用域中有一个全局对象`window`

在全局作用域中所有的变量都会作为`window`对象的属性保存

在全局作用域中的所有函数都会作为`window`对象的方法保存

### 变量声明提前

变量的声明提前。使用var关键字声明变量，会在所有代码执行之前声明（但是不会赋值，赋值还需执行到代码）

函数的声明提前。使用**函数声明形式**创建的函数，会在代码执行之前就被创建。使用**函数表达式形式**创建的函数，不会被声明提前，所以不能再声明提前创建。 

### 函数作用域

在函数中使用变量，如果函数为声明则会往上一层作用域找，直到找到全局作用域，如果都没有找到会报`ReferenceError`错误。每一次调用函数都会创建一个作用域，并且是独立互不影响的。

在局部函数中访问全局变量可以使用window对象。

在函数中，不使用var关键字声明的变量都会成为全局变量。

### this

解释器在每次调用函数的时候都会传入一个隐含参数，这个隐含参数就是`this`对象，该对象称为函数的上下文对象，根据函数的调用方式不同，this指向的对象也不同：以函数形式调用时，this永远都是window；以对象方法形式调用时，this是对象本身。

### 跳过若干

### 构造函数

构造函数和普通函数的区别就是调用方法不同，构造函数需要使用`new`关键词创建，并且构造函数的返回类型为**Object**类型。构造函数名通常为首字母大写。构造函数执行流程：

1. 立刻创建一个新对象
2. 将新对象（新建的那个对象）设置为对象中的this *
3. 逐行执行代码
4. 将新对象作为返回值

`instanceof`可以用来检查一个对象是否是一个类的实例。所有的对象都是Object的后代

### 原型对象

我们创建的每一个函数，构造器都会向方法中添加一个属性`prototype`，这个属性对应一个对象，这个对象就是**原型对象**。如果函数作为普通函数调用，原型对象没有任何作用；如果作为构造函数调用，会为该构造函数创建一个唯一的原型对象。

原型对象相当于一个类的公共区域，这个类的所有实例都能访问这个原型对象。不同的类拥有不同的原型对象。我们可以将类共有的属性或方法添加到其中。

```javascript
function Person(){
    
}
Person.prototype.a=123;

var per = new Person();
//per.__proto__ == Person.prototype
console.log(per.a); //123
```

如果访问一个对象中的属性或方法，它会首先在自身中寻找，如果未找到则会去原型对象中寻找。原型对象也有原型，这是一个**原型链**。

### 垃圾回收

把不用的对象设置为`null`。~~详细以后填坑~~

### call和apply

`call()`和`apply()`是函数对象的一个调用方法，这两个方法都会调用函数执行。可以将一个对象指定为第一个参数，则该对象成为函数执行时的this。

其他区别：call方法可以将实参在对象之后依次传递；apply方法需要将参数封装到一个数组中传递

``` js
function func(a,b){
    console.log(a);
    console.log(b);
}
func.call(obj,2,3);

func.apply(obj,[2,3]);
```

### argument对象

调用函数时，浏览器每次都会传递两个隐含参数

1. 函数上下文对象this
2. 封装实参的`argument`对象

argument对象是一个类似数组的对象，可以通过索引操作，函数调用时的所有实参都会在argument中保存，即使不定义形参，也可以通过argument对象使用实参。

``` javascript
function fun(){
    console.log(argument instanceof Array); //false
    console.log(argument[1]); //true
    console.log(argument.lenght); //2
    console.log(argument.callee) //会打印当前正在执行的函数对象
}

fun('hello',true);
```

> callee属性可以留意一下

## DOM

document object model