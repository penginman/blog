---
title: "Vue学习笔记"
date: 2022-03-07T18:16:47+08:00
draft: false
---

## 理解MVVM

Vue参考的MVVM模型

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/16466499314121646649931302.png)


M：模型(Model) ：data中的数据
V：视图(View) ：模板代码
VM：视图模型(ViewModel)：Vue实例
观察发现：

1. data中所有的属性，最后都出现在了vm身上。
2. vm身上所有的属性及Vue原型上所有属性，在Vue模板中都可以直接使用。

## 数据代理

### 回顾Object.defineProperty方法

```vue
let number = 18
object.defineProperty(person,'age',{
 value:18,
 enumerable:true，//控制属性是否可以枚举，默认值是false
 writable:true，//控制属性是否可以被修改，默认值是false
 configurable:true //控制属性是否可以被删除，默认值是false
//当有人读取person的age属性时，get函数（getter)就会被调用，且返回值就是age的值
get(){
	return number
},
//当有人修改person的age属性时，set函数(setter)就会被调用，且会收到修改的具体值
set(value){
	number = value
}
})
```

### 数据代理

何为数据代理？通过一个对象来修改另一个对象。

例如，obj={x:100}

obj2={y:200}

使用Object.defineProperty方法，设置obj2的x属性get和set方法与obj绑定

```vue
Object.defineProperty(obj2,'x',{
get(){
return obj.x
},
set(value){
obj.x = value
}
})
```

在script标签里设置的data属性值，绑定的是vm中的_data属性

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/16467060352831646706034381.png)

**Vue中的数据代理：**
通过vm对象来代理data对象中属性的操作（读/写）
**Vue中数据代理的好处：**
更加方便的操作data中的数据
**基本原理：**
通过0bject.defineProperty()把data对象中所有属性添加到vm上。为每一个添加到vm上的属性，都指定一个getter/setter。在getter/setter内部去操作（读/写）data中对应的属性。

## 事件处理

事件的基本使用：

1. 使用**v-on:xxx** 或 **@xxx** 绑定事件，其中xxx是事件名；
2. 事件的回调需要配置在methods对象中，最终会在vm上；
3. methods中配置的函数，不要用箭头函数！否则this就不是vm了；
4. methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；
5. @click="demo" 和 @click="demo($event)"效果一致，但后者可以传参；
5. @click="yyy"，其中yyy可以写一些简单的语句 

## 事件修饰符

使用示例

```html
<a href="xxx" @click.prevent="showInfo">
```

event事件中可以使用e.preventDefault()阻止默认事件，vue中可以使用@click.**prevent**='xxx'的修饰方式使用该方法。

**扫盲：**

事件冒泡 ：当一个元素接收到事件的时候 会把他接收到的事件传给自己的父级，一直到window 。（注意这里传递的仅仅是事件 并不传递所绑定的事件函数。所以如果父级没有绑定事件函数，就算传递了事件 也不会有什么表现 但事件确实传递了。）

事件捕获和事件冒泡：DOM2级事件’规定的事件流包含3个阶段，**事件捕获阶段、处于目标阶段、事件冒泡阶段**。首先发生的事件捕获为截获事件提供机会，然后是实际的目标接收事件，最后一个阶段是事件冒泡阶段，可以在这个阶段对事件做出响应。https://www.cnblogs.com/christineqing/p/7607113.html

![](https://images2017.cnblogs.com/blog/1174211/201712/1174211-20171201225153933-1205737719.png)



**Vue中的事件修饰符：**
1.prevent：阻止默认事件（常用）；
2.stop：阻止事件冒泡（常用）；
3.once：事件只触发一次（常用）；
4.capture：使用事件的捕获模式（捕获阶段就执行函数）；
5.self：只有event.target是当前操作的元素时才触发事件；
6.passive：事件的默认行为立即执行，无需等待事件回调执行完毕（比如scroll和wheel事件，wheel回调函数很麻烦的时候，可以使用passive优先执行滚轮默认行为）；

## 键盘事件

使用方法示例

```vue
<input @keydown.enter="showInfo">
```

* Vue中常用的按键别名：
  回车 => enter
  删除 => delete（捕获“删除”和“退格）
  退出 => esc
  空格 => space
  换行 => tab（特殊，必须配合keydown去使用）
  上 => up
  下 => down
  左 => left
  右 => right
* Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要keyab-case（驼峰命名改为短横线命名）
* 系统修饰键（用法特殊）：ctrl、alt、shift、meta

1. 配合keyup使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。
2. 配合keydown使用：正常触发事件。

* 也可以使用keyCode去指定具体的按键（不推荐）
* Vue.config.keyCodes.自定义键名 = 键去定制按键别名

### 小tips

修饰符可以连续写比如`@click.prevent.stop`和`@keydown.ctrl.y`

## 计算属性

vue中绑定的数据修改时，vue会重新解析模板。

使用示例

```
computed{
 fullName:{
 	get(){
 		return firstName + lastName
 	}
 }
}
```



计算属性**定义**：根据**已有的属性**进行一些计算加工生成的新属性就叫计算属性。使用配置项`computed`，计算属性最终会在`vm`对象身上，但是不在`_data`里。

原理：底层借助了Objcet.defineproperty方法提供的gtter和setter。

计算属性的值使用getter调用，多次调用会使用**缓存**，getter什么时候调用？

1. 初次读取计算属性时
2. 所依赖的数据发生改变时

如果计算属性要被修改，必须使用set函数响应修改，切记set函数中要修改依赖的属性

### 简写

确定只读取不修改可以使用简写模式。示例

```
fullNmae(){
	return 
}
```

## 监视属性

配置对象`watch:{}`，代码示例

```
watch：{
	isHot:{
		immediate:true，//初始化时让handler调用一下
		//handler什么时候调用？当isHot发生改变时。
		handler(newValue,oldValue){
			console.log("isHot被修改了',newValue,oldValue)
			}
		}
	}
```

监视属性也可以监视**计算属性**，监视属性的另一种写法

```
vm.$wathc('变量名',{配置项同上

})
```

## 深度监视

Vue中的watch默认不监测对象内部值的改变，配置`deep:true`可以监测对象内部值的改变。Vue自身可以监测对象内部值的改变，但是Vue提供的watch默认不可以，使用watch的时候根据数据的具体结构，决定是否采用深度监视。

有简写形式

## 监视属性vs计算属性

还是要看具体需求。

1. computed能完成的功能，watch都可以完成。
2. watch能完成的功能，computed不一定能完成，例如：watch可以进行异步操作。

两个重要的小原则：

1. 所被Vue管理的函数，最好写成普通函数，这样this的指向才是vm 或 组件实例对象。
2. 所有不被Vue所管理的函数（定时器的回调函数、ajax的回调函数等），最好写成箭函数，这样this的指向才是vm或组件实例对象。

## 绑定样式

1. class样式
   写法：class="xxx" xxx可以是字符串、对象、数组。
   字符串写法适用于：类名不确定，要动态获取。
   对象写法适用于：要绑定多个样式，个数不确定，名字也不确定。
   数组写法适用于：要绑定多个样式，个数确定，名字也确定，但不确定用不用。
2. style样式
   :style="{fontSize：xxx}"其中xxx是动态值。
   :style="[a,b]"其中a、b是样式对象。

## 条件渲染

`v-if`写法：
(1).v-if="表达式"
(2)v-else-if="表达式"
(3).v-else="表达式"
适用于：切换频率**较低**的场景。
特点：不展示的DOM元素直接被移除。
注意：v-if可以和:v-else-if、v-else一起使用，但要求结构不能被“打断”。

`v-show`写法：v-show="表达武"
适用于：切换频率**较高**的场景。
特点：不展示的DOM元素未被移除，仅仅是使用样式隐藏掉。

备注：使用v-if的时，元素可能无法获取到，而使用v-show一定可以获取到。

还有一个点if和template配合使用保持原来的html结构

## 列表渲染

v-for指令：

1. 用于展示列表数据
2. 语法：v-for="(item,index)in xxx'":key="yyy"
3. 可遍历：数组、对象、字符串（用的很少）、指定次数(用的很少)

key的作用和原理

面试题：react、vue中的key有什么作用？(key的内部原理)

1. 虚拟DoM中key的作用：
   key是虚拟DoM对象的标识，当数据发生变化时，Vue会根据**新数据**生成**新的虚拟DoM**,随后Vue进行**新虚拟DoM**与**旧虚拟DoM**的差异比较，比较规则如下：

   对比规则：

   1. 旧虚拟DoM中找到了与新虚拟DoM相同的key。

      若虚拟DoM中内容没变，直接使用之前的真实DoM！

      若虚拟DoM中内容变了，则生成新的真实DoM,随后替换掉页面中之前的真实D0M。

   2. 旧拟DoM中未找到与新虚拟DoM相同的key

      创建新的真实DOM,随后渲染到到页面。

2. 用index作为key可能会发的问题：
   若对数据进行：逆序添加、逆序明除等破坏顺序操作，会产生没有必要的真实DoM更新==>界面效果没问题，但效率低。

   如果结构中还包含输入类的DoM，会产生错误DoM更新==>界面而有问题。

3. 开发中如何选择key？
   最好使用每条数据的唯一标识作为key，比如id、手机号、身份证号、学号等唯一值。
   如果不存在对数据的逆序添加、逆序则除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的。

![](https://img.braindance.top/article/2022/06/15/684dd822cdc7d26d5e587df6a93c9a0d.png) 

## 列表排序&过滤

能用computed的就不用watch

## 监测数据的原理

### 对象的检测

首先对data数据进行加工，设置响应式的getter和setter，大概的核心代码如下

```javascript
let data = {
  name:"xx",
  address:"dd"
}

const obs = new Observer(data)

let vm={}
vm._data = data = obs

function Observer(obj){
  //汇总对象里的属性形成一个数组
  keys = Object.keys(obj)
  //遍历
  keys.forEach(k => {
    Object.defineProperties(this,k,{
      get(){
        return obj[k]
      },
      set(val){
        //Vue重新模板解析
        obj[k] = val
      }
    })
  });
}
```

并且Vue底层使用递归的方式对所有嵌套的对象属性都进行了数据监测方法的加工。

Vue提供了一个API的set方法`Vue.set()`或`vm.$set()`，可以为已经创建的对象添加响应式的属性。传入参数列表：target，key，val。局限性：**target对象不能是Vue实例或者Vue实例的根数据对象**。

### 数据的检测

通过**包裹**数组中更新元素的方法实现，本质上做了两件事

1. 调用原型中的数组更新方法
2. Vue重新进行模板解析，更新页面

Vue包裹的7种方法：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

也可以使用Vue提供的API：`Vue.set()`或`vm.$set()`

## 收集表单数据

根据`<input>`标签的类型不同，v-model收集的值也不同，分为以下几种情况

* text类型。收集的是value属性
* radio类型。收集的同样是value属性，但是需要给标签配置value值并成组
* checkbox类型。如果没有配置value属性，收集的是选项框的`checked`属性。如果配置了value属性，根据v-model绑定属性的初始值不同有两种情况：1.绑定属性初始值为**非数组**，收集的是checked属性。2.绑定属性为数组，收集的就是value组成的数组

v-model的三个常用修饰符：

* lazy。输入框失去焦点时收集数据
* number。输入字符串为有效数字
* trim。过滤首尾空格

## 过滤器

对数据进行格式化显示。语法：

注册过滤器：全局注册Vue.filter(name,callback) 或 局部注册new Vue{filters{}}

使用过滤器：{{xxx | filter}} 或 v-bind:属性 = xxx | filter

过滤器可以接受多个参数，第一个默认是管道符前的数据。多个管道符可以串联（按顺序调用）

## 内置指令

v-bind：单项绑定解析表达式可以简写为`:xxx`

v-model：双向绑定数据

v-for：遍历数组/字符串/对象

v-on：绑定事件监听，可以简写为`@`

v-if（v-else）：条件渲染

v-show：控制节点是否展示

v-test：所在节点渲染文本内容。和插值语法区别，前者会直接替换掉节点里的内容，插值语法不会。

v-html：向所在节点渲染包含html结构的内容。指令可以识别html结构。但是要注意XSS攻击
