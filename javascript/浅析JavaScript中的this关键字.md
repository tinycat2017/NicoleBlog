# 浅析JavaScript中的this关键字

###  导语

不得不说，作为一名初级的前端开发者，this关键字这个问题对于我来说一直是一个痛点，什么是this？什么是函数的执行环境？
函数的执行环境和this之间的关联是什么？以及在不同的函数调用方式(function invocation,method invocation，constructor invocation，indirect invocation)里this的具体值是什么？
于是我带着这些问题对this关键字进行了深入的学习，并写了一个相关的demo。

##  一. 认识JavaScript函数的执行环境

以前学C语言的时候，学过函数的执行过程，在C语言中，函数调用的时候，会在内存里的程序执行栈上push即将被调用的函数的地址入栈，然后再根据被调用的函数的地址，执行函数里相应的代码。
我发现这种调用方式对应于JavaScript的函数调用方式也是有类似的地方，每一个函数都有自己的执行环境，当执行流进入到这个函数时，该执行环境会被推入到环境栈中，函数执行完毕之后，推出该执行环境，把执行流控制权还给之前的执行环境。

每个执行环境都有一个与之关联的变量对象，环境中所定义的所以变量以及方法都保存在这个变量对象中。
全局执行环境是一个最外围的执行环境，根据js所处在不同的宿主环境来决定全局执行环境，在web浏览器中，全局执行环境是window对象，全局里所定义的所以变量和方法都在这个对象里。

##  二. 认识JavaScript中的函数的调用方式

由于 this 关键字和JavaScript中函数的调用方式有着很密切的关系，所以我们先谈谈JavaScript中的函数的调用方式。

在JavaScript中，函数的调用一共有四种方式：

* 函数调用(function invocation)

```js
function add(a,b) {
  return a + b;
}
//函数调用(function invocation)
var result = add(1, 2);
```

值得注意的是，立即调用函数(IIFE)也是属于函数调用这种方式。

```js
var calculate = (function (num) {
  return 10 + num;
})(2);
```

* 方法调用(method invocation)

```js
var addNum = {
  sum:	function (a,b) {
      		return a + b;
  			}
}
//方法调用(method invocation)
addNum.sum(1, 2);
```

* 构造函数调用(constructor invocation)

```js
function Car(name,price) {
  this.name = name;
    this.price = price;
}
//构造函数调用(constructor invocation)
var Bus = new Car('bus', 100000);
```

