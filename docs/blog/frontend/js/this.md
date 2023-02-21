# this指向

为了看出this指向，首先需要知道this的绑定方式

1. 默认绑定
2. 隐私绑定
3. 硬绑定
4. new绑定

## 四种绑定方式

### 默认绑定

   ```js
    function sayName(){
      console.log(this.name);
    }
    var name = 'curry';
    fn(); // curry
   ```

   默认绑定是在不能应用其它绑定规则时使用的默认绑定规则，通常是独立的函数调用<br>
   在上面调用sayName()时，使用的是默认绑定，this在非严格模式下指向全局对象也就是window，而在严格模式下，this指向undefined，undefined上没有this对象，因此会抛出错误

### 隐式绑定

   ```js
    function sayName(){
      console.log(this.name);
    }

    var obj = {
      name: "curry",
      sayName: sayName
    }
    obj.sayName(); // curry
   ```

   隐式绑定的形式通常是xxx.fn()，函数的调用是在某个对象上触发的，它的调用位置上存在上下文对象<br>
   虽然sayName函数声明在外部，但在调用sayName时，它的调用位置会使用obj的上下文来引用函数，隐式绑定会把函数调用中的this绑定到这个上下文对象<br>
   需要注意的是，对象属性链中只有最后一层会影响到调用位置

   ```js
    function sayName(){
      console.log(this.name);
    }

    var playerB = {
      name:'curry',
      sayName: sayName
    }

    var playerA = {
      name: 'paul',
      friend: playerB
    }

    playerA.friend.sayName(); // curry
   ```

   只有最后一层会确定this指向的是什么，因此在使用隐式绑定的时候，只需要关注最后一层即可

### 隐式绑定存在绑定丢失问题

```js
  function sayName(){
      console.log(this.name);
  }
  var player = {
    name: 'curry',
    sayName: sayName
  }
  var name = 'james';
  var say = player.sayName;
  say(); // james
```

say函数指向了sayName的引用，在调用的时候，与player对象没有关系，它的上下文对象是window对象，因此在判定是否为隐式绑定的时候，只需要记住格式xxx.fn()即可，如果没有xxx，那么肯定不是隐式绑定

### 回调函数中的隐式绑定丢失

```js
  function sayName(){
    console.log(this.name);
  }
  var playerA = {
    name: 'curry',
    sayName: function(){
        setTimeout(function(){
            console.log(this.name);
        })
      }
  }
  var playerB = {
    name: 'paul',
    sayName: sayName
  }
  var name= 'james';
  playerA.sayName();
  setTimeout(playerB.sayName,100);
  setTimeout(function(){
    playerB.sayName();
  },200);
```

结果是

```js
  james
  james
  paul
```

* 第一条输出语句很好理解，setTimeout的回调函数中，this使用默认绑定，因此this指向window对象
* 第二条输出语句中，可以理解为setTimeout(fn,100); playerB.sayName赋值给了fn，此时fn中的this与playerB对象没有任何关系，又因为setTimeout回调函数中的this使用默认绑定，因此this指向window对象
* 第三条输出语句，使用了隐式绑定，this指向playerB。

### 硬绑定

   ```js
   var playerName = {
      name:"james",
      sayName:function(){
        console.log(this.name);
      }
    }

    var playerA = {
      name:"curry"
    }

    var playerB = {
      name:"paul"
    }

    playerName.sayName.call(playerA); // curry
    playerName.sayName.call(playerB); // paul
   ```

硬绑定是通过call、apply、bind方法，指定this所指向的对象<br>
call,apply和bind的第一个参数，就是对应函数的this所指向的对象。call和apply的作用一样，只是传参方式不同。call和apply都会执行对应的函数，而bind方法不会。

### 硬绑定仍然存在绑定丢失的问题

```js
  function sayName(){
    console.log(this.name);
  }
  var playerA = {
    name: 'curry',
    sayName: sayName
  }
  var name = 'paul';
  var say = function(fn) {
    fn();
  }
  say.call(playerA, playerA.sayName); 
```

在这个例子中，虽然是使用了call方法实现了硬绑定，但是在参数传递的时候，playerA.sayName赋值给了fn变量，没有指定this的值，因此使用的是默认绑定

* 有解决办法吗？ 当然有，只需要在调用fn的时候，也将它进行硬绑定

```js
  function sayName(){
    console.log(this.name);
  }
  var player = {
    name: 'curry',
    sayName: sayName
  }
  var name = 'paul';
  var say = function(fn) {
    fn.call(this);
  }
  say.call(player, player.sayName); 
```

此时，输出的结果为: paul，因为player被绑定到say函数中的this上，fn又将这个对象绑定给了sayName的函数。这时，sayName中的this指向的就是player对象。

### 硬绑定的例外

当将null或者时undefined作为this绑定对象传入call、apply、bind，这些值在调用的时候都会被忽略，实际使用的是默认绑定规则

```js
  function sayName(){
    console.log(this.name);
  }

  var name = 'curry';
  var obj = {
    name: 'paul'
  };

  sayName.call(null); // curry
```

输出curry，因为此时使用的默认调用

### new绑定

javaScript和Ｃ＋＋不一样，并没有类，在javaScript中，构造函数只是使用new操作符时被调用的函数，这些函数和普通的函数并没有什么不同，它不属于某个类，也不可能实例化出一个类。任何一个函数都可以使用new来调用，因此其实并不存在构造函数，而只有对于函数的“构造调用”。

   ```js
   function Player(name){
      this.name = name;
      console.log(this.name);
    }

    var p1 = new Player("curry"); // curry
   ```

**在使用构造函数时，使用new关键字实例化对象会执行以下操作**

1. 在内存中创建一个新对象
2. 这个新对象的内部 [[Prototype]] 特性被赋值为构造函数的prototype属性：

```js
  p1.__proto__ = Player.prototype
```

3. 构造函数内部的this被赋值给新的对象 也就是新对象和函数调用的this会绑定起来

```js
  Player.call(p1,'curry');
```

4. 执行构造函数内部的代码 即给新对象添加属性和方法

```js
  p1.name;
  console.log(p1);
```

5. 如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象

**因此，在使用new来调用函数的时候，会将新对象绑定到这个函数的this上**

## 绑定的优先级

在知道了this的四种绑定规则后，我们需要知道如果出现同时多种绑定规则，它们的优先级应该是什么

new绑定 > 硬绑定 > 隐式绑定 > 默认绑定

## 箭头函数

首先需要明确箭头函数没有this，箭头函数的this继承的是外层代码块的this，箭头函数也无法用call、apply、bind这些方法来改变this的指向

```js
  var obj = {
      hi: function(){
          console.log(this);
          return ()=>{
              console.log(this);
         }
      },
      sayHi: function(){
          return function() {
              console.log(this);
              return ()=>{
                  console.log(this);
              }
          }
      },
      say: ()=>{
          console.log(this);
      }
  }
  let hi = obj.hi();  //输出obj对象
  hi();               //输出obj对象
  let sayHi = obj.sayHi();
  let fun1 = sayHi(); //输出window
  fun1();             //输出window
  obj.say();          //输出window
```

分析一下执行结果

1. obj.hi()使用的是隐式绑定，this绑定在obj上，因此输出obj
2. hi()执行的是箭头函数，箭头函数继承的外层代码块的this，外层代码块的this绑定的obj，因此输出obj对象
3. sayHi()发生了隐式绑定丢失的问题，因此此时是默认绑定，this指向window对象
4. fun1()执行的是箭头函数，外层代码块的this指向window对象
5. obj.say()执行的是箭头函数，外层代码块的this指向的window对象

## 总结

**如何准确判断this的指向**

1. 判断是否使用new绑定，如果是，this绑定的是新创建的对象
2. 判断是否使用硬绑定，即是否使用call、apply、bind方法，如果是，this绑定的是指定的对象，注意如果call、apply、bind方法传入的参数是null或者是undefined，则使用默认绑定
3. 判断是否使用隐式绑定，隐式绑定的格式通常是xxx.fn()，this指向的xxx对象
4. 如果都不是，则使用默认绑定
5. 判断执行的是否是箭头函数，箭头函数没有this，它里面的this继承的是外层代码块

## 三个面试题

第一题

```js
    function a(){
      function b(){
        console.log(this);
        function c(){
          'use strict'
          console.log(this);
        }
        c()
      }
      b()
    }
    a()
```

第二题

```js
    var name = "小白";

    function special(){
      console.log("姓名" + this.name);
    }

    var girl = {
      name:"小红",
      detail:function(){
        console.log("姓名" + this.name);
      },
      woman:{
        name:"小黄",
        detail:function(){
          console.log("姓名" + this.name);
        },
      },
      special:special
    }

    girl.detail(); 
    girl.woman.detail();
    girl.special(); 
```

第三题

```js
    var name = "小红";
    
    function a(){
      var name = "小白";
      console.log(this.name);
    }

    function d(i){
      return i();
    }

    var b = {
      name:"小黄",
      detail:function(){
        console.log(this.name);
      },
      bibi:function(){
        return function(){
          console.log(this.name);
        }
      }
    }

    var c = b.detail;
    b.a= a;
    var e = b.bibi();
    a();
    c();
    b.a();
    d(b.detail);
    e();
```
