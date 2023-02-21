# 运行原理与作用域

在了解作用域的概念之前，我们首先来聊一聊浏览器的工作原理

## 浏览器的工作原理

![image-20211127230129415](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127230129415.png)

当你打开一个浏览器之后，会经历如下过程：

1. 向服务器请求index.html这个文件
2. 遇到link标签后，向服务器请求获取到css文件
3. 遇到script标签后，向服务器请求获取到js文件

## 浏览器的内核

接着我们来简单认识一下浏览器的内核

浏览器内核称为**排版引擎**（layout engine），也称为**浏览器引擎**（browser engine）、**页面渲染引擎**（rendering engine）

或**样版引擎**。

常见的浏览器内核有哪些？

1. Gecko：早期被Netscape和Mozilla Firefox浏览器浏览器使用；

2. Trident：微软开发，被IE4~IE11浏览器使用，但是Edge浏览器已经转向Blink；

3. Webkit：苹果基于KHTML开发、开源的，用于Safari，Google Chrome之前也在使用；

4. Blink：是Webkit的一个分支，Google开发，目前应用于Google Chrome、Edge、Op

## 浏览器的渲染过程

如图我们可以看到，过程大致是：浏览器内核解析html文件转换成DOM树，同时解析css文件，然后根据css规则，将他们附加在一起，生成一棵渲染树，进行layout（布局）之后进行绘制，最后展示到显示器上。

> 为什么要进行layout，是因为浏览器处于不同宽度的时候的的样式是不同的，因此需要layout根据浏览器的不同状态进行布局

![image-20211127193930366](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127193930366.png)

你可能会说：等等，js哪去了？

其实，在执行浏览器渲染过程的时候，HTML解析的时候遇到了script标签后会停止解析HTML，而去加载和执行JavaScript代码。但是JavaScript代码又由谁来执行呢？答案是：**JavaScript引擎**

## JavaScript引擎

有其他编程语言经验的同学应该知道，高级语言都需要转换成0101的二进制代码来执行的，而JavaScript同样属于高级语言，因此对于JavaScript来说，无论是交给浏览器还是Node来执行，最终都会转换成二进制代码。

但是这个过程是由谁来转换的呢，其实就是JavaScript引擎

**常见的JavaScript引擎**：

1. SpiderMonkey：第一款JavaScript引擎
2. JavaScriptCore：WebKit中的JavaScript引擎，苹果公司开发
3. V8引擎：谷歌公司开发

## 浏览器内核和JavaScript引擎之间的关系

* 浏览器内核：负责HTML解析、布局、渲染等等相关的工作；

* JavaScript引擎：解析、执行JavaScript代码；

了解了这个之后我们接着来看看强大的V8引擎

## V8引擎

先来看看官方的定义

> V8是用C ++编写的Google开源高性能JavaScript和WebAssembly引擎，它用于Chrome和Node.js等。
>
> 它实现ECMAScript和WebAssembly，并在Windows 7或更高版本，macOS 10.12+和使用x64，IA-32， ARM或MIPS处理器的Linux系统上运行。
>
> V8可以独立运行，也可以嵌入到任何C ++应用程序中

下面这张图是V8引擎的执行原理图

![image-20211127200540869](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127200540869.png)

V8引擎大概会做如下的事情

1. Blink（浏览器内核的一种）把js下载下来，将js代码交给V8引擎，
2. Stream模块获取到代码后进行编码转换，交给Scanner
3. Scanner会进行**词法分析**，将代码转换成tokens
   * 比如 ```const name = 'curry'``` ，生成的tokens就会将这些信息分析成：```tokens:[{type:'keyword',value:'const'},{type:'identifier',value:'name'}]```
4. tokens经过PreParser和Parser后转换成AST抽象语法树**（语法分析）**
   * 其中Parser就是直接将tokens转换成AST树
   * PreParser称为**预解析**，为什么需要预解析，是因为并不是所有js代码都会在一开始被执行，如果直接对所有的js代码进行解析，必然会影响网页的运行效率，所以V8引擎实现了**延迟解析（Lazy-Parsing）**的方案，只解析暂时需要的内容，而对函数的全部解析是在**函数被调用的时候才会进行**
   * 比如一个outer函数里面有一个inner函数，此时这个inner函数就会被预解析
5. 生成AST树后，会被一个叫**Ignition**的模块转换成字节码，然后就是代码的执行过程了
6. V8引擎内部还有一个叫做**TurboFan**的编译器，可以将字节码编译为CPU可以直接运行的机器码
   1. 如果一个函数被多次调用就会被标记为**热点函数**，那么就会经过TurboFan编译为优化的机器码，提高代码的运行效率
   2. 但是机器码也会被转换为字节码，这是因为如果一个函数的参数的**类型发生了变化**，之前的机器码就不能正确运算，就会被逆向转换成为字节码

## JavaScript的执行过程

通过下面这个例子，我们来把JavaScript的执行过程弄清楚，别急，只有弄清楚了JavaScript的执行原理，才能从根本上弄懂作用域

```js
var age = 123
function foo(){
    console.log(age)
}

var num1 = 1
var num2 = 1
var result = num1 + num2
console.log(result)

foo()
```

### 初始化全局对象

js引擎在执行代码前，会在堆内存中创建一个全局对象：GlobalObject（后面统称为GO）

该对象在所有的作用域中都可以访问到，里面存在JavaScript的内置函数，比如Date、Array、setTimeout等

其中还有一个**window属性**指向GO对象自己

![image-20211127203416475](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127203416475.png)

### 执行上下文栈

V8为了执行代码会创建一个叫**执行上下文栈（Execution Context Stack，简称ECS）**，它是用于执行**代码的调用栈**

我们一开始执行的是全局的代码，因此为了让全局代码能够顺利执行，需要创建**全局执行上下文（Global Execution Context，简称GEC）**

创建完GEC后会被放入到ESC中，

此时GEC会包含两部分内容：VO（variable object）和需要执行的代码，（其中VO指向GO）。

执行代码前，在经过parser被转换成AST的过程中，会将定义的全局变量和函数加入到GO中，但并不会赋值，这就是**变量提升**

![image-20211127205657896](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127205657896.png)

之后再一步步执行代码，进行赋值操作，此时有函数的话，将会在堆内存中为函数对象（FunctionObject）开辟一段空间，然后让GO中的函数的变量指向函数对象的内存地址。

![image-20211127210323753](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127210323753.png)

### 遇到函数将会怎样执行

在执行过程中遇到一个函数的时候，会根据函数体创建一个函数执行上下文（Functional Execution Context，后面统称为FEC），压入到ECS中

**FEC包含三部分**：

1. 解析函数成为AST时，会创建一个Activation Object（AO）对象，AO对象中包含形参、arguments、函数定义和指向函数对象、定义的变量
2. 作用域链：由VO（在函数中也就是AO）和父级的VO组成，查找时会一层层往上找，这就是**作用域链的查找方式**，因此这也说明，作用域链是在解析的时候就已经确定了
3. this绑定的值：执行时绑定

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20211127223451753.png)

## 作用域的经典面试题

```js
var n = 100
function foo1(){
    console.log(n) // ?
}

function foo2(){
    var n = 200
    console.log(n) 
    foo1()
}

foo2()
console.log(n)
```

仔细分析后，我们可以明确得出其他两个n的值，但是有注释的那一行的输出究竟是什么

当foo2（）调用后，会创建一个函数执行上下文，里面的VO指向**foo2的AO对象**，此时foo2的作用域链应该是**VO+GO**，然后调用foo1。

此时foo1（）会创建一个函数执行上下文，里面的VO指向**foo1的AO对象**，此时foo1的作用域链应该是**VO+GO**。

因此当它去查找n这个变量的时候，没有从VO中找到n，就会去parent scope也就是GO中查找n这个变量，进而输出100这个值

## 变量环境和记录

最后我们再来聊一聊变量环境和记录

实际上在最新的ECMA规范中，它对一些词汇进行了修改

来对比一下早期的ECMA规范和最新的ECMA规范

早期的：

> 每一个执行上下文会被关联到一个变量环境（**variable Object**）中，在源代码中的变量和函数的声明会被作为**属性**添加到VO中。
>
> 对于函数来说，参数也会被添加到VO中。

最新的：

> 每一个执行上下文会关联到一个变量环境（**variable Environment**），在执行代码中变量和函数的声明会作为**环境记录**（Environment Record）添加到变量环境中。
>
> 对于函数来说，参数也会被作为环境记录添加到变量环境中。

通过这个变化我们可以知道，在最新的规范中，VO对象已经被叫做了VE对象，而变量和函数的声明原来被称为属性现在被称为了环境记录

## 总结

通过认识浏览器和V8引擎，我们一步步了解了JavaScript的运行原理，从而理解了作用域链到底是怎么查找的，以及作用域链其实是在解析的时候就已经确定了，与函数的调用位置无关。最后通过面试题，加深了作用域的理解，相信看完这篇文章，不管遇到什么样的关于作用域链的面试题都能轻松搞定。
