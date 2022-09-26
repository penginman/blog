---
title: "Python-Day-100 笔记"
date: 2022-06-06
lastmod: 2022-09-24
draft: false
---
## 函数

### 函数的参数

可以使用可变参数`def func(*args)`，参数前面的`*`代表`args`是一个可变参数。

### 用模块管理函数

每个文件代表一个模块，不同模块的函数命名可以相同，但是如果如下代码引用

```python
from module1 import foo
from module2 import foo
foo()
```

程序会调用最后一个调用的`foo`函数。
如果导入模块中除了定义的函数以外有可执行的代码，则Python解释器在导入这个模块时就会执行这些代码。可以利用只有被Python解释器直接执行的模块的名字才是 **\_\_main\_\_** 进行判断

```python
def foo():
    pass
# __name__是Python中一个隐含的变量它代表了模块的名字
# 只有被Python解释器直接执行的模块的名字才是__main__
if __name__ == '__main__':
    print('call foo()')
    foo()
```

在其他模块再导入上述模块时，if条件中的语句就不会执行

### 变量作用域

Python查找一个变量时会按照“局部作用域”、“嵌套作用域”、“全局作用域”和“内置作用域”的顺序进行搜索。内置作用域及python内置的标识符如：`input`、`print`、`int`。
可以使用`global`关键字来指示局部函数中的变量来自**全局变量**，`nonlocal`关键字表示变量来自**外部嵌套函数内的变量**

> 事实上，减少对全局变量的使用，也是降低代码之间耦合度的一个重要举措，同时也是对[迪米特法则](https://zh.wikipedia.org/zh-hans/%E5%BE%97%E5%A2%A8%E5%BF%92%E8%80%B3%E5%AE%9A%E5%BE%8B)的践行。

## 字符串和常用数据结构

### 字符串

反斜杠`\`用来转义。
`\`后面可以跟八进制和六进制来表示字符，`\u68d2`使用unicode字符编码表示字符。
如果不想使用`\`转义，可以在字符串前加上字母`r`说明

```python
s1 = r'\n\\hello, world!\\\n'
print(s1) # \n\\hello, world!\\\n
```

使用`+`进行字符串拼接，使用`*`重复一个字符串的内容，`in`、`not in`来判断字符串中是否包含子串，`[]`、`[:]`用来切片运算。

字符串类型是一种结构化的、非标量类型，所以会有一系列的属性和方法。
字符串对象身上的**常用的函数**：
* len()；获取字符串长度
* capitalize()；字符串首个字母大写的拷贝
* title()；每个字符串单词首字母大写的拷贝
* upper()；所有字符串大写的拷贝
* find()；查找字串位置。未找到值为 -1
* startswith()；以某字符开始。类似的有endswith()
* center(50,'\*')；将字符串以指定宽度居中，填充指定字符。类似的有rjust()、ljust()
* isdigit()；是否为数字构成
* isalpha()；是否为字母构成
* isalnum()；是否为数字字母构成
* strip()；获取处理左右两侧空格后的拷贝

**格式化输出字符串**
使用`%d`、`%s`等占位符，并在字符串结尾使用`%()`列表对应。
字符串提供的`format()`方法。
Python 3.6以后提供了语法糖表示，在字符串前使用`f`。
```python
a,b = 5,10
print('%d * %d = %d' % (a, b, a * b))
print('{0} * {1} = {2}'.format(a, b, a * b))
print(f'{a} * {b} = {a * b}')
```

### 列表
数值类型（int、float）是标量类型，列表（list）是一种结构化的、非标量类型。
可以使用下标的方式遍历列表元素，或者for循环遍历列表元素
```python
list = [1,2,3,4,5,6]
//通过下标访问
for index in range(len(list)):
	print(list[index])
//遍历元素访问
for elem in list:
	print(elem)
//使用enumerate()处理获得元素及其下标
for index, elem in enumerate(list1): 
	print(index, elem)
```

> enumerate() 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。

列表也可以使用`+`进行拼接，使用切片操作获得列表的复制。
`sorted()`函数返回列表进行排序后的备份，不会影响原来的列表，我们设计函数应该像 sorted 函数一样不产生副作用。可以设置关键字`reverse=True`使列表逆置，关键字`key=len`根据字符串长度进行排序

### 生成式和生成器
生成式表达式创建列表容器和生成器创建列表容器
```python
//用列表的生成表达式语法创建列表容器
//表达式生成的容器里面元素已经准备就绪，所以会耗费较多内存空间
f1 = [x ** 2 for x in range(1, 1000)]

//下面的代码创建的不是一个列表而是一个生成器对象
//生成器会在使用时，经过运算获取数据，不占用额外空间，但是消耗时间
f2 = (x ** 2 for x in range(1, 1000))
```

还可以使用关键字`yield`将一个普通函数改造成**生成器函数**，下面是使用生成器函数打印斐波那契数列的例子
```python
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
        yield a


def main():
    for val in fib(20):
        print(val)


if __name__ == '__main__':
    main()
```

### 元组
元组与列表类似也是一种容器数据类型，但是元素内的元素不可以被修改。
可以使用列表的方法访问元素，不能修改元组元素，但是可以引用新的元组，原来的元素就会被垃圾回收。
使用`list()`函数将元组转换成列表，使用`tuple()`函数将列表转换成元组。
为什么要用元组？
1. 元组可以在多线程环境中保证对象状态不会被修改，一个不变对象自动就是线程安全的，这样就可以省掉处理同步化的开销。
2. 元组在创建时间和占用的空间上面都优于列表。

### 集合
集合跟数学上的集合是一致的，不允许有重复元素，而且可以进行交集、并集、差集等运算。
如果定义集合时有重复元素会被剔除；集合可以添加删除元素；
集合的成员进行交集（&）、并集、差集等运算。例
```python
set1 = {1, 2, 3, 3, 3, 2}

# 集合的交集、并集、差集、对称差运算
print(set1 & set2)
# print(set1.intersection(set2))
print(set1 | set2)
# print(set1.union(set2))
print(set1 - set2)
# print(set1.difference(set2))
print(set1 ^ set2)
# print(set1.symmetric_difference(set2))
# 判断子集和超集
print(set2 <= set1)
# print(set2.issubset(set1))
print(set3 <= set1)
# print(set3.issubset(set1))
print(set1 >= set2)
# print(set1.issuperset(set2))
print(set1 >= set3)
# print(set1.issuperset(set3))
```

### 字典
字典可以存储任何数据类型，每一个元素都为一个键一个值的`key:value`格式。
```python

dic1={'筑基丹': 1100, '元灵精华': 900, '磐龙宝剑': 11000}

# 创建字典的构造器语法
items1 = dict(one=1, two=2, three=3, four=4)

# 通过zip函数将两个序列压成字典
items2 = dict(zip(['a', 'b', 'c'], '123'))

# 对字典中所有键值对进行遍历
for key in dic1:
    print(f'{key}: {dic1[key]}')
	
# get方法也是通过键获取对应的值但是可以设置默认值
print(scores.get('神农鼎', 60))

```


## 面向对象编程
比较正式的说法

> "把一组数据结构和处理它们的方法组成对象（object），把相同行为的对象归纳为类（class），通过类的封装（encapsulation）隐藏内部细节，通过继承（inheritance）实现类的特化（specialization）和泛化（generalization），通过多态（polymorphism）实现基于对象类型的动态分派。"

python中使用`class`关键字定义类，然后在类中定义属性和方法
```python
class Student(object):

    # __init__是一个特殊方法用于在创建对象时进行初始化操作
    # 通过这个方法我们可以为学生对象绑定name和age两个属性
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def study(self, course_name):
        print('%s正在学习%s.' % (self.name, course_name))
		
def main():
	stu1 = Student('大卫',18)
	stu1.study()
```

在python中可以给对象的属性或方法添加访问权限，就像Java中的私有属性，可以在属性或方法前添加双下划线`__`表示改方法是私有的，并且调用对象的属性或方法时会报`'XXX' object has no attribute '__xx'`的错误。
但是Python并没有从语法上保护属性或方法的私密性，如果知道更换名字的规则仍然可以访问到它们，如
```Python
class Test:

    def __init__(self, foo):
        self.__foo = foo

    def __bar(self):
        print(self.__foo)
        print('__bar')
		
def main():
    test = Test('hello')
    test._Test__bar()
    print(test._Test__foo)

if __name__ == "__main__":
    main()
```


之所以这样设定，可以用这样一句名言加以解释，就是"We are all consenting adults here"。因为绝大多数程序员都认为开放比封闭要好，而且程序员要自己为自己的行为负责。实际开发过程中Python程序员会遵循一种命名习惯就是让属性以单下划线开头表示属性是受保护的，其他类访问这样的属性时应该慎重，这种做法并不是语法上的规则，而是一种隐喻。
作者踩过的坑[《Python - 那些年我们踩过的那些坑》](http://blog.csdn.net/jackfrued/article/details/79521404)。

**面向对象的支柱** ：**封装**、**继承**、**多态**。

封装的感觉有点像webapi


## 面向对象进阶

### @property装饰器

之前讲过的使用单下划线标识变量为私有的，不建议外界直接访问，如果想访问可以使用 **@property装饰器** 可以用来包装getter和setter方法
```python
class Person(object):
	def __init__(self,name,age):
		self.name = name
		self.age = age
	@property
	def name(self):
		return self.name
	
	@property
	def age(self):
		return self.age
	
	@age.setter
	def age(self,age):
		self.age = age

#没有定义name的setter方法，如果访问的话会报AttributeError: can't set attribute错误
```

### \_\_ slots \_\_魔法
由于Python是一种动态语言，允许在程序运行中给对象或属性绑定新的方法和属性，可以使用\_\_slots\_\_魔法来限定类可以绑定的属性和方法。	\_\_slots\_\_是一个变量，英文名直接翻译过来的意思是插槽，可以理解为在其中的方法或属性规定为类可拥有的
```python

class Person(object):
	
	__slots__ = ('_name', '_age', '_gender')
	
	def __init__(self,name,age):
		self.name = name
		self.age = age
	@property
	def name(self):
		return self.name
	
	@property
	def age(self):
		return self.age
	
	@age.setter
	def age(self,age):
		self.age = age

#如果添加或设置不在__slots__包含的属性，如'_is_man'则会报错AttributeError: 'Person' object has no attribute '_is_man'
```

### 静态方法和类方法
类中的方法不一定都是对象方法（给类对象传递消息），也有一些通用的**静态方法**，如：三角形类中还为创建对象之前需要判断三条边是否能组成三角形，判断边能否组成三角形就可以设置为静态方法。**类方法**个人理解类似于Java中的默认构造方法。
静态方法使用`@staticmethod`装饰器，举例：
```python
class Triangle(object):

    def __init__(self, a, b, c):
        self._a = a
        self._b = b
        self._c = c

    @staticmethod
    def is_valid(a, b, c):
        return a + b > c and b + c > a and a + c > b

def main():
	#静态方法
	a,b,c = 6,8,10
	if Triangle.is_valid(a, b, c):
		print("构成三角形")

if __name == '__main__':
	main()
```

类方法使用`@classmethod`装饰器，类方法的第一个参数约定名为`cls`参数，它代表的是当前类相关的信息的对象，举例：
```python

class Clock(object):
	
    """数字时钟"""

	def __init__(self, hour=0, minute=0, second=0):
		self._hour = hour
		self._minute = minute
		self._second = second
		
	@classmethod
	def now(cls):
		ctime = localtime(time())
		return cls(ctime.tm_hour, ctime.tm_min, ctime.tm_sec)
	
	def show(self):
        """显示时间"""
        return '%02d:%02d:%02d' % \
               (self._hour, self._minute, self._second)

	
def main():
	clock = Clock.now()
	clock.show()


if __name == '__main__':
	main()
```

### 类之间的关系
类之间的关系主要有三种：is-a、has-a和use-a关系
* is-a关系。通常称为继承或者泛化，如学生和人的关系，狗和宠物的关系。
* has-a关系。通常称为关联，如部门和公司、汽车和引擎的关系。
	* 关联关系如果是整体和部分的关联，则称这种关联为**聚合关系**
	* 如果整体负责了部分的生命周期（比如我和我的脑子）整体和部分是不可分割的，整体的生命周期结束就表示部分的生命周期结束，这种关联是最强的关联关系，称为**强聚合**或者**组合关系**
* use-a关系。称为依赖关系，表示一个类依赖于顶一个类的定义，依赖关系总是单向的。比如某人要过河，需要借用一条船，此时人与船之间的关系就是依赖。

这些都属于UML（统一建模语言）用来描述对象之间的关系。参考内容：[UML的四种关系](https://blog.csdn.net/weixin_45920385/article/details/121315813)。还可以去看《UML面向对象设计基础》一书

### 继承和多态
我们可以在已有的类的基础之上建立新的类，可以直接将原有类中的属性和方法继承下来，从而减少重复代码。继承中有两个概念：
* 父类。也成超类、基类，是提供类继承信息的类
* 子类。也成派生类、衍生类，是得到继承信息的类

子类除了可以继承父类中的属性和方法，还可以定义自己新的属性和方法，所以子类的功能也会更全面。在开发中也经常使用子类替换父类对象，这一规则称为[里氏替换原则](https://zh.wikipedia.org/wiki/%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99)。
继承的例子：
```python 
class Person(object):
	def __init__(self, name, age):
		self._name = name
		self._age = age
		
	"""
	省略setter和getter方法
	"""
class Student(Person):
	
	def __init__(self, name, age, grade):		
		super().__init__(name, age)#调用父类的初始化方法
		self._grade = grade

```
子类继承父类方法以后，可以对父类已经实现的方法重新构造，这种行为称为方法重写（override）。通过重写父类的方法，可以使得不同的子类拥有不同的行为，这就是所谓的**多态**。
```python
from abc import ABCMeta, abstractmethod


class Pet(object, metaclass=ABCMeta):
	"""宠物"""

	def __init__(self, nickname):
		self._nickname = nickname

	@abstractmethod
	def make_voice(self):
		"""发出声音"""
		pass
	
class Dog(Pet):
	def make_voice(self):
		print('%s: 汪汪汪...' % self._nickname)

class Cat(Pet):
	def make_voice(self):
		print('%s: 喵...喵...' % self._nickname)
```
在上面代码中，创建了 `Pet` 抽象类，抽象类就是不能够被创建成对象的类，它是专门用来让其他类继承它。Python中没有Java那样对抽象类的支持，但是可以使用 `abc` 模块的 `ABCMeta` 元类和 `abstractmethod` 包装器来达到抽象类的效果。`Dog` 和 `Cat` 类分别继承了抽象类 `Pet` 并重写了 `make_voice` 方法，当分别创建两个对象并调用方法时，这个方法就表现出了多态行为。

## 文件和异常

开发中经常需要将数据持久化存储，最简单的方式就是存到文件中，Python可以使用 `open` 函数，指定文件名、操作模式、编码信息等获取到操作文件的对象。具体的操作模式如下表

| 操作模式 | 具体含义                         |
| -------- | -------------------------------- |
| `'r'`    | 读取 （默认）                    |
| `'w'`    | 写入（会先截断之前的内容）       |
| `'x'`    | 写入，如果文件已经存在会产生异常 |
| `'a'`    | 追加，将内容写入到已有文件的末尾 |
| `'b'`    | 二进制模式                       |
| `'t'`    | 文本模式（默认）                 |
| `'+'`    | 更新（既可以读又可以写）         |

下图是来自菜鸟教程中的辅助图记。

![文件操作](https://www.runoob.com/wp-content/uploads/2013/11/2112205-861c05b2bdbc9c28.png)

### 读写文本文件

读取文件时使用 `r` 操作模式即可，`encoding` 参数可以指定编码（若未指定，默认值是None，读取时会使用系统默认编码），如果编码不一致会无法解码导致读取失败。读取文件时会遇到各种错误如：文件找不到会引发 `FileNotFoundError`，指定了未知的编码会引发 `LookupError`，而如果读取文件时无法按指定方式解码会引发 `UnicodeDecodeError`，可以使用 `try-except` 代码块捕获异常
```python
def main():
    f = None
    try:
        f = open('致橡树.txt', 'r', encoding='utf-8')
        print(f.read())
    except FileNotFoundError:
        print('无法打开指定的文件!')
    except LookupError:
        print('指定了未知的编码!')
    except UnicodeDecodeError:
        print('读取文件时解码错误!')
    finally:
        if f:
            f.close()


if __name__ == '__main__':
    main()
```
 `try-except` 代码块最后可以使用 `finally` 块执行外部资源的释放的操作，因为在它其中的代码，不论程序正常还是异常执行都会被执行，即使是 `sys` 的 `exit` 函数都会被执行，因为 `exit` 函数实质上是引发了 `SystemExit` 异常。
 除了 `finally` 块来执行外部资源的释放以外，还可以使用 `with` 关键字指定文件对象的上下文环境并在离开上下文环境时自动释放文件资源。

 ```python
	with open('致橡树.txt', 'r', encoding='utf-8') as f:
		print(f.read())

 ```

除了使用 `read` 方法读取文件内容以外，还可以使用 `for-in` 循环逐行读取或是 `readlines` 将文件按行读取到一个列表容器中。
```python
with open('致橡树.txt', 'r', encoding='utf-8') as f:
	print(f.read())
	
ith open('致橡树.txt', mode='r') as f:
	for line in f:
		print(line, end='')
		
with open('致橡树.txt') as f:
	lines = f.readlines()
	print(lines)
```

如果想要写入文件可以使用 `w` 操作模式，但是其会覆盖文件原来的内容，如果想要追加内容可以使用 `a` 操作模式
### 读写二进制文件
二进制文件常见的有图片的复制或写入图片内容，需要使用二进制模式 `b`，可以接合读或写模式为 `rb` 、`wb`。
### 读写JSON文件

一个JSON数据类型和Python数据类型的对照表

|                JSON |       Python |
|---------------------|--------------|
|              object |         dict |
|               array |         list |
|              string |          str |
| number (int / real) |  int / float |
|        true / false | True / False |
|                null |         None |


Python 中的 `json` 模块包含了对 JSON 格式字串操作的函数，分别是：

* `dump` - 将 Python 对象按照 JSON 格式序列化到文件中
* `dumps` - 将 Python 对象序列化为 JSON 格式字串
* `load` - 和 dump 相反。(将文件中的 JSON 数据反序列化为 Python 对象)
* `loads` -和 dumps相反。（将字符串的内容反序列化成Python对象）

```python
mydict = {
  "tieba":{
    "type": "image",
    "container": [
      { "icon": "< img src =\"emoji/tieba/呵呵.png\">", "text": "呵呵" },
      { "icon": "< img src =\"emoji/tieba/哈哈.png\">", "text": "哈哈" }
	]
    }
		}
with open('data.json', 'w', encoding='utf-8') as fs:
		json.dump(mydict, fs)
```

留个异常的坑以后来填。[总结：Python中的异常处理](https://segmentfault.com/a/1190000007736783)