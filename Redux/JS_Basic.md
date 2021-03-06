Javascript 杂谈 ：熟悉基本概念

<!-- TOC -->

- [1. 信息、变量、数据类型和变量对象](#1-信息变量数据类型和变量对象)
  - [1.1. 看不见的信息](#11-看不见的信息)
  - [1.2. 客观的数据](#12-客观的数据)
  - [1.3. 变量只是一个名字](#13-变量只是一个名字)
  - [1.4. 类型是数据的结构](#14-类型是数据的结构)
  - [1.5. 变量对象](#15-变量对象)
  - [1.6. 作用域链](#16-作用域链)
- [2. 面向对象](#2-面向对象)
  - [2.1. this (strict)](#21-this-strict)

<!-- /TOC -->

# 1. 信息、变量、数据类型和变量对象

ECMAScript 对于变量的定义及语法，借鉴了众多语言的特性，同时也形成了自己独特的运作机制。松散的变量和简单的语法让编程过程更加惬意自由。

## 1.1. 看不见的信息

众所周知，计算机是用来处理信息的工具，而程序就是处理信息的步骤。但是信息是看不见摸不到的，如何进行操作？我们首先使用符号来承载它，也就是经常提到的数据，但是这样好像也不行，例如在黑漆漆的房间里我画下了若干符号，你依然无法感知啊。所以，要有光，利用信号让你感知数据和信息。比如这篇文章在我的脑中，你是无法感知的，我用汉语（符号）将它写出，然后通过光传到你的眼睛里；你的大脑将光信号再翻译回汉语（符号），然后分析它的信息。

好了，请描述一下 “20 ” 这个信息。什么我已经描述了，你看到的 “20” 就是这个信息的阿拉伯数字符号表示。那么你是怎么看到这个符号的？真正的过程可能比较复杂：

- 我在电脑上输入 “20” （符号），通过 ASCII 转化成了 0、1格式（另一种符号）。
- 我的网卡将 0、1 转化成电平（信号），通过网线传到你的网卡。
- 你的网卡重新将电平翻译回 0、1格式，然后根据 ASCII 转化为你屏幕上的 “20” 。

在上述过程中是否可以保存信号来间接的存储数据和信息？当然可以存储设备都是这样工作的，晶体管的稳态表示 1，光盘的凹凸坑对应了 0 、1 。我们把 “20” 存在内存中，终于把信息从看不见摸不到的状态变成了实际存在且能保存的物理信号。那么程序是如何读取内存中的 “20” 的？

## 1.2. 客观的数据

用哲学的概念来引述，存储在内存中的 “20”，它是具体存在的，是客观的。而将它映射到程序中的就是变量，它是逻辑的（主观的）。

![map.png](http://upload-images.jianshu.io/upload_images/5518635-2cdc5e207b16123d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.3. 变量只是一个名字

变量在 ECMAScript 中是所谓的标识符，也就是一个名称或名字。它的作用只是用来指示某个物体或对象，**变量没有数据**。比如每个人都有名字，但是名字里含有你的数据吗？名字只是用来辨识人，同理变量只是用来分辨具体数据或对象的。

你可能会问 ```var a = 20``` 中的 “20” 不就是一个标识符吗？这条语句的具体步骤应该是：

- 人类输入熟悉的标识符 “20” 
- 编译器将 “20” 编译成相应类型的数据存储在内存中
- 使用变量 a 来映射内存中的数据

也就是说这里的 a 和 20 都是内存中数据的标识符，20 是人类熟悉的标识符，通常出现在初始化过程，a 则是程序使用的标识符，并且因为是逻辑的，所以可以变动，这也是称为变量的原因。当然在 ECMAScript 中除了 20 这样的数字，还定义多种数据类型。

## 1.4. 类型是数据的结构

在人类世界中国际标准的基本单位有 7 个：长度m，时间s，质量kg，热力学温度（Kelvin温度）K，电流单位A，光强度单位cd（坎德拉），物质的量mol。这些基本单位可以推出物理世界的所有物理量。而在 ECMAScript 中定义了 6 种数据的类型，并且不支持自定义类型，因此所有值都是这 6 中数据类型之一。它们又分为两类，第一类称为基本数据类型，包含：```Undefined```， ```Null```， ```Boolean```， ```Number```， ```String```。另外一类称为复杂数据类型 ```Object``` ，它是由多个无序的键值对组成。

根据数据类型的分类， ECMAScript 中的变量可能映射两种不同的类型值，基本数据类型值和引用类型值。基本数据类型值就是该类型的数据，引用类型值则是该类型对象的引用，它们都和变量存储在内存栈空间，而引用类型的对象存储在内存的堆空间。

![variable.png](http://upload-images.jianshu.io/upload_images/5518635-9ace56e352888094.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大体上可以认为引用类型就是继承自 ```Object``` 复杂数据类型的一种数据结构，它包含了数据和与数据相关的操作。常用的有 ```Function```， ```Array```， ```Date```， ```RegExp``` ，而这些类型数据在声明时就已经转化成对象存在内存中，只有在没有引用情况下会被垃圾收集。除此之外还有一种特殊的引用类型，就是基本包装类型，在说明基本包装类型前先了解一下什么是数据类型，《数据结构》中定义了 “数据类型 = 数据元素 + 关系集 + 关系集的基本操作” ，也就是说数据类型不仅仅包含数据还有和数据相关的操作。

```js
// 程序中声明一个 ```string``` 类型数据元素，但是注意并没有定义相关的操作
var str = 'some text'
var listItem = [1, 2, 3]
// 而在这里使用了```substring``` 方法，
// 也就是说 ECMAScript 将 'some text' 这个数据元素自动转化成某种数据类型的数据存储在内存中，随后释放。
var sub_str = str.substring(2)   // 'me text'
var subList = listItem.slice(2)  // [3]
```

在语句 ```var str = 'some text'``` 中变量 ```str``` 映射的是一个基本数据类型的值，注意仅仅是数值，但是为什么可以调用 ```substring()``` 方法？这就是基本包装类型的功能，基本包装类型同样继承自 ```Object``` ，但是数据元素都定义为 ```Number``` 这种基本数据类型，同时给出一些相关的基本操作，它包含有 ```Number```， ```String```， ```Boolean``` 。是的和基本类型的样子一模一样，这样的黑箱效应让程序员无须留意转化过程的。程序员看到的是 ```'some text'```，而程序在调用它时转化成一种 ```String``` 基本包装类型的对象实例，但是基本包装类型特殊处就是只在调用时转化，调用结束后释放。

```js
// var sub_str = str.substring(2) 的等效程序
var str = new String('some text')
 // 因为此时 str 已经是基本包装类型了，拥有了 substring 方法
var sub_str = str.substring(2)  
// 最后释放这个对象数据
str = null
```

可将上述内容总结为：数据赋值过程中需要判断数据是基本数据类型还是引用类型，基本数据类型的数据直接赋值给变量，而引用类型的数据将引用赋值给变量。

下面通过例子来巩固学习内容：

```js
var a = 20
var b = a
b = 30
console.log(a)
```

第一条语句所赋值的数据是 ```Number``` 基本数据类型，那么只需要将值映射给变量 a 即可。

第二条语句首先调用了变量 a ，将数值 20 复制给变量 b 映射的数值中，此时变量 b 所映射的数值与变量 a 所映射的数值存在不同的内存单元中。这里可能需要补充一个概念，在 ECMAScript 中赋值和参数传递都是传值操作，也就是将一个变量映射的值传给另一个变量映射的值。或者简单的认为变量和映射的值是一个整体。

![copy.png](http://upload-images.jianshu.io/upload_images/5518635-17e25c8abe3f2fcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三条语句将基本数据类型的数值 30 映射给变量 b 。

再看一个例子：

```js
var c = [1, 2, 3]
var d = {x: 10, y: 20, z: 30}
var e = c
e[2] = 4
var f = d
f.y = 40

// c: [1, 2, 4], e: [1, 2, 4]
// d: {x: 10, y: 40, z: 30}, f: {x: 10, y: 40, z: 30}
```

上例中，c 是 ```Array``` 引用类型， d 是 ```Object``` 引用类型，它们的值都是可变的。而更需要说明的是 e 和 c 同时指向一个对象， d 和 f 也是，当 e 和 f 被修改时，c 和 d 的值也随之发生变化。

![referrence.png](http://upload-images.jianshu.io/upload_images/5518635-ae2ed2ea26b0f815.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.5. 变量对象

在之前的图示中，已经多次描述内存的操作方式，即栈和堆。所谓的栈是一种对内存单元的操作方式，使得内存对外呈现出一种特殊的数据结构：堆栈。堆栈本身是一种受限的线性表，其受限主要表现在对数据的操作位置只能是栈顶，拥有先进后出（FILO）的特点。现实中收纳乒乓球的盒子就是一个堆栈。在程序中通常利用堆栈解决嵌套调用的问题，例如下面的程序：

```js
function sum(a, b) {
  return a + b
}

var result = sum(1, 2)
```

当程序开始执行，编译器会为此创建一个全局执行环境又称为执行上下文，以后每进入一个函数，都会创建相应的执行环境，并依次将执行环境推入内存的栈空间。在执行环境中最重要的两个部分，一个是变量对象，它用来保存在环境中所定义的变量和函数，另一个是作用域链（后面章节说明）。

接下来通过图示解释上述程序的执行过程，在程序开始执行时，全局执行环境被压入栈空间：

![global_variable_object.png](http://upload-images.jianshu.io/upload_images/5518635-ec10e2f836c8539c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当程序调用 ```sum``` 函数时，sum 函数的执行环境被压入栈空间：

![sum_AO.png](http://upload-images.jianshu.io/upload_images/5518635-f119598332045160.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然执行完 sum 函数后，sum 函数的执行环境就会从栈空间中弹出，把控制权交还给之前进入栈空间的执行环境（此例中就是全局执行环境）。

之前已经学习到变量和其映射的值都保存在内存的栈空间中，现在明白了其中的原因，它们作为执行环境中的变量对象压入进执行环境栈。此时仔细思考会发现另外一个与变量对象相关的问题，那就是一个执行环境中的程序是否可以访问另外一个执行环境的变量对象？这个问题的答案就是作用域链。

## 1.6. 作用域链

在执行某一个环境中的程序时，会创建一个变量对象的作用域链。它的作用是给出该执行环境中的程序能够有权访问的变量和函数的有序链表。作用域链的首个结点始终是当前执行环境的变量对象，下一个结点来自包含的外部环境，依此类推直到全局执行环境。全局执行环境的变量对象总是作用域链的最后一个结点。

请看下面的示例代码：

```js
function interestRate (x) {
  return function yearBalance (y) {
    return y * (1 + x)
  }
}

var currentDeposit = interestRate(0.03)
var balance = currentDeposit(10000)
console.log(balance)
```

下图根据定义给出了各执行环境的作用域链，为了清楚的展现之间的关系将变量对象从执行环境中分离。

![scope_chain.png](http://upload-images.jianshu.io/upload_images/5518635-81c26b45515e770e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

程序解析变量是沿着作用域链一级一级的搜索，搜索过程始终从作用域链的前端开始，然后逐级向后回溯，直到找到变量为止。
上例中 yearBalance 函数通过作用域链访问到了 interestRate 函数的变量 x 。

最后我们来疏通概念间彼此纠缠的关系，执行环境是一个函数在内存中的投影，变量对象是函数的内部数据，而作用域链是函数暴露的接口。每次执行函数，首先将执行环境投影到内存，然后根据作用域链获取参数，执行函数代码并修改内部数据也就是变量对象，这里对于可变的变量对象我们又称之为活动对象。期间忽略了许多细节，如活动对象和作用域链的构建，垃圾收集等。

## 1.7 变量对象与活动对象的区别

两者都指向一个对象这是大家的共识，在一些文章中将两者的区别归结于执行环境的不同阶段，文中都把执行环境的生命周期看成两个阶段，分别是 **创建阶段** 和 **执行阶段** ，变量对象处于创建阶段而活动对象处于执行阶段。 这是没有问题的，但是这些文章中对创建阶段的概念还是比较模糊的，个人认为创建阶段所指的是，执行环境从开始到销毁除了执行阶段外的所有生命周期。比如上例中如果进入 ```yearBalance``` 执行环境的执行阶段，那么 ```interestRate``` 执行环境就处在创建阶段。


# 2. 面向对象

## 2.1. this (strict)

在经典的面向对象语言中，this 只会出现在类中，作为该类实例（对象）的占位符使用。 ECMAScript 部分继承和实现了上述规则，唯一例外之处正是 ECMAScript 中最美妙最古怪的**函数**。之前学习到所有的函数都是 ```Function``` 引用类型的实例（对象），按说应该没有 this 这个占位符，但是有一类函数起到了类的作用，可以生成对象，这就是构造函数。因此在函数中又引入了 this 占位符，这与经典定义格格不入，出现了对象中有另一个对象占位符的混乱概念，使得 this 在函数中晦涩难懂。

我们从最基本的定义分析，以此梳理 this 的使用规则。

- 除函数外，this 不会出现在对象中。

  ```js
  // 对象字面量
  var o = {
    a: 1,
    b: this.a + 1
  }  // Object {a: 1, b: NaN}

  // 数组
  var arr = [1, this, 3] // [1, Object, 3]
  ```
- 箭头函数是匿名函数，不会担任构造函数的角色，因此没有 this 。

- 函数中的 this 是在函数被执行时才确认。

  回顾之前的内容，函数在运行时创建执行环境，执行环境中包含变量对象和 this 对象，变量对象是在执行代码过程中形成，this 对象在执行开始获取并无法修改。

- 构造函数的 this 来自它所创造的实例。

- 非构造函数的 this 是引用它的对象。

```js
function Person (name, age) {
  this.name = name
  this.age = age
  this.sayName = sayName
}
function sayName () {
  console.log(this.name)
}

var alien = {
  name: 'alien',
  age: 1000,
  sayName: sayName
}
var tom = new Person('tom', 22)

alien.sayName()
tom.sayName()
```

上述只是 this 最基本的判断方法，如果仔细分析作用域同时结合闭包思想，可以尽可能的减少 this 的出错率。