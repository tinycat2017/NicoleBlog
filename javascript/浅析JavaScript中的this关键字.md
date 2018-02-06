# 浅析JavaScript中的this关键字

### 导语

不得不说，作为一名初级的前端开发者，this关键字这个问题对于我来说一直是一个痛点，什么是this？什么是函数的执行环境？
函数的执行环境和this之间的关联是什么？以及在不同的函数调用方式(function invocation,method invocation，constructor invocation，indirect invocation)里this的具体值是什么？
于是我带着这些问题对this关键字进行了深入的学习，并写了一个相关的demo。

## 一. 认识JavaScript函数的执行环境

以前学C语言的时候，学过函数的执行过程，在C语言中，函数调用的时候，会在内存里的程序执行栈上push即将被调用的函数的地址入栈，然后再根据被调用的函数的地址，执行函数里相应的代码。
我发现这种调用方式对应于JavaScript的函数调用方式也是有类似的地方，每一个函数都有自己的执行环境，当执行流进入到这个函数时，该执行环境会被推入到环境栈中，函数执行完毕之后，推出该执行环境，把执行流控制权还给之前的执行环境。

每个执行环境都有一个与之关联的变量对象，环境中所定义的所以变量以及方法都保存在这个变量对象中。
全局执行环境是一个最外围的执行环境，根据js所处在不同的宿主环境来决定全局执行环境，在web浏览器中，全局执行环境是window对象，全局里所定义的所以变量和方法都在这个对象里。

## 二. 认识JavaScript中的函数的调用方式

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

* 间接调用(indirect invocation)

```js
function increment(num) {
	return ++num;
}
//间接调用(indirect invocation)
increment.call(undefined,1);
```

## 三. 在四种JavaScript函数调用方式中的this

### 1. 在函数调用中的this

一般来说，函数调用的时候this的值就是全局环境执行对象，也就是如果JavaScript当前的执行环境是浏览器的话，这个时候的函数调用中的this就是指的window对象。

```js
function add(a,b) {
	console.log(this === window)//true
	return a + b;
}
//函数调用(function invocation)
var result = add(1, 2);
```

但是需要注意的一点是，在严格模式下，函数调用中的this的值就不再是全局环境执行对象了，而是undefined。

在函数调用中有一个很需要注意的地方，内层定义的函数的this并不一定等同于外层定义的函数的this，
也就是说如果外层定义的函数是以函数声明的方式表达的，那么内层定义的函数的this还是和外层函数的this一样，都是全局环境执行对象。

```js
function add(a,b) {
	alert(this === window);//true
	function innerf() {
		alert(this === window);//true
	}
	innerf();
	return a + b;
}
//函数调用(function invocation)
var result = add(1, 2);
```

但是，当外层函数是以对象的属性定义在对象里时，外层函数的this为当前对象，而内层函数中的 this 是window(严格模式下为 undefined)。

```js
var addNum = {
	sum: function (a,b) {
		alert(this === addNum)//true
		function innerf() {
			alert(this === window)//true
		}
		innerf();
		return a + b;
	}
}
//方法调用(method invocation)
addNum.sum(1, 2);
```

所以，在这种情况下，内层函数是无法通过this对象来读取到addNum对象里的属性的。
不过，为了能解决这个问题，我们可以使用在内层函数调用的时候调用内层函数的call函数来将外层函数的this值传入到内层函数中，也就是调用innerf.call(this)。

### 2. 在方法调用中的this

当我们以方法调用的模式调用函数的时候，函数内部的this的值是调用这个函数的对象，
在执行方法调用的时候，被调用的函数的执行环境也就是调用这个函数的对象，
需要注意的一点是，当一个JavaScript对象从它的原型那里继承了方法时，这个对象调用从其原型继承的函数的时候，
这个从原型那里继承来的函数执行环境仍然是这个对象(而不是其原型对象)，即这个函数的this值为当前调用此函数的对象。

```js
function Test1() {

}
Test1.prototype.addNew = function () {
	alert(this == temp)//true
}
var temp = new Test1();
temp.addNew();
```

在ES6的class语法中，被创建的对象也是其内部方法中的this对象的值。

这一切看起来似乎顺理成章，但是我们可以想象一个情况，就是当我们将对象内部的函数抽取出来并将之赋值到一个新的变量上，
那么当我们再通过调用这个变量所指向的函数时，此时函数内部的this还会是之前的那个对象吗？

```js
var addNum = {
	sum: function (a,b) {
		alert(this === addNum)// ????
		return a + b;
	}
}
var newFunc = addNum.sum;
newFunc(1, 2);
```

实际上，结合我之前所说的，函数调用和方法调用的形式是不同的，根据这点我们就可以知道这个问题的答案了，
虽然我们将对象内部的函数赋值给了一个新的变量，但是当我们调用这个变量来执行函数的时候，是以函数调用的方式来执行的，所以此时函数内部的this值应该是全局环境执行对象(严格模式下为undefined)。
现在让我们再把问题更深入一步，该怎样实现把一个对象内的函数抽取出来赋值给了一个新的变量的时候，并在调用这个变量去执行函数时，这个函数内部的this值仍然是这个对象？
答案很简单，通过函数自身的绑定方法，也就是bind方法，将对象的值赋给函数的this。

```js
var addNum = {
	sum: function (a,b) {
		alert(this === addNum)// true
		return a + b;
	}
}
var newFunc = addNum.sum.bind(addNum);
newFunc(1, 2);
```

### 3. 在构造函数调用中的this

说到在构造函数调用中的this，我们需要先清楚构造函数的概念，在JavaScript中函数虽然是对象，但是函数也能够去创建新的对象，而通常用函数去创建对象的形式是在类似于函数调用的方式前加上new关键字。
而用new关键字来创建一个对象一共会经历四步：

1. 创建一个Object对象实例

2. 将构造函数的执行环境设置为新创建的这个实例

3. 执行构造函数中的代码

4. 返回新生成的对象实例

所以，在构造函数调用的时候，this关键字就是当前被构造出来的新对象。
也就是说构造函数调用时的执行环境也就是当前被构造出来的新对象。

```js
function City() {
	alert(this instanceof city);//true
	this.name = 'beijing';
}
var bj = new City();
alert(bj.name); //beijing
```

在ES6的class语法中，constructor方法的作用是负责对象的初始化，在constructor方法里this关键字的值就是新创建出来的那个对象。

```js
class City {
	constructor (){
		alert(this instanceof City);//true
		this.name = 'beijing';
	}
}
var bj = new City();
```

在JavaScript中我们也可以不使用new关键字来创建对象，我们可以使用函数调用的方式来创建一个对象，只要在函数的最后加上 `return  创建的对象` 即可。
但是这种创建对象的方式可能会带来一个问题，例如下面的代码示例：

```js
function City() {
	alert(this instanceof City);//false
	alert(this === window);//true
	this.name = 'beijing';
	return this;
}
var bj =  City();
```

理由正是之前提到的函数调用的执行环境是全局环境执行对象，所以在浏览器上，由于函数调用的执行环境是window，
函数内部的this值即为window对象，所以这样的调用方式并不会产生一个新的对象，而是给全局执行对象增加了新的属性。

### 4. 在间接调用中的this

我们知道在JavaScript中函数也是对象，所以函数也拥有对象的特性，即函数也可以有自己的属性，
因此函数也拥有一些内部方法，比如`.call()` ,`.apply()`。
这两个函数的内部方法的共同特点是它们接受的第一个参数即被当做函数的执行环境对象，不同点是`.call()`接受的第一个参数之后的参数形式为list，而`.apply()`接受的第一个参数之后的参数形式为array。

```js
function increment(num) {
	return ++ num;
}
increment.call(window,1); // 2
increment.apply(window, [1]);// 2
```

正如代码示例，当以`increment.call()`，`increment.apply()`的形式调用函数的时候，其实是向increment函数的执行环境对象赋值并执行函数的代码。

因此我们可以知道出在像`.call()`，`.apply()`这样的间接调用中，函数中的this关键字的值即为这两个间接调用函数的第一个参数值。

```js
function increment(num) {
	alert(this === newOb)//true
	return ++ num;
}
var newOb = {name:'addFunc'};
increment.call(newOb,1); // 2
increment.apply(newOb, [1]);// 2
```

由于有了间接调用的方式，我们可以利用间接调用来使得函数的执行环境对象为我们所指定的值。

### 5. Bound function

现在来讲一讲在JavaScript函数中的另一种调用——— Bound function，熟悉JavaScript的人都知道，
JavaScript的函数对象还拥有另一个方法——— `.bind()`，这个方法能够产生一个和调用这个方法的原函数一样代码的新函数，并把在调用时传入的第一个参数作为这个新函数的执行环境对象。
也就是说这个函数和`.call()`，`.apply()`这两个方法不一样的是执行`.call()`，`.apply()`这两个间接调用时，原函数会被立刻执行，
而`.bind()`是产生并且返回一个和原函数一样代码的函数对象，且这个新函数的执行环境对象是`.bind()`方法的第一个参数，
但是这个新的函数并不会被立即执行，也就是说这个新的函数对象只是被预定义了执行环境对象而已。

```js
var caculateObj = {
	numbers:[1,2,3],
	getNumbers: function () {
		return this.numbers;
	}
}
var bindFunc = caculateObj.getNumbers.bind(caculateObj);
bindFunc();// [1,2,3]
var simpleFunc = caculateObj.getNumbers;
simpleFunc();// window or undefined
```

在调用`.bind()`的时候需要注意的是，这种方式的绑定this对象是“永久”的，也就是说通过调用`.bind()`方法定义了新函数内部的this值是会一直存在并且不可被再次修改的，
即便是在对新函数使用`.call()`，`.apply()`也是不能够重新定义this值的。
不过我们可以对调用`.bind()`创造出来的新函数进行构造函数调用来改变this值，不过这种方式并不推荐~

### 6. 在箭头函数中的this关键字

想必接触过ES6的人对箭头函数一定不陌生，那么在箭头函数里的this值是什么呢？

对于箭头函数来说，它本身是不能够像普通函数一样创建自己的执行环境对象的，但是它可以继承它的外部函数中的执行环境对象的，也就说其实对于箭头函数来说它的this值是来自于它外部函数的this值。
仔细一想，这种方式是不是又提供了之前提到的在对象内部的函数属性里定义新的函数，新的函数的this为window对象或者undefined的问题的一种解决办法呢？
之前如果我们想让对象内部的函数属性里定义的新的函数拥有指向该对象的this关键值是需要通过`.bind()`方法来实现的，而现在我们可以通过箭头函数来做到这些。

```js
var caculateObj = {
	numbers:[1,2,3],
	getNumbers: function () {
		alert(this === caculateObj);//true
		var insideFunc = () =>{
			alert(this === caculateObj);//true
		}
		insideFunc();
		return this.numbers;
	}
}
caculateObj.getNumbers();
```

而且箭头函数内部的this值一旦被定义后是不能被修改的，即便调用`.call()`，`.apply()`也是不能够修改的，
而且箭头函数也不能够通过构造函数调用来改变this值，因为箭头函数并不能够作为构造函数。

在使用箭头函数的时候需要注意一个小细节，那就是箭头函数内部的this值一定是其外部函数的this值，所以箭头函数被定义的地方是很重要的，比如下面这个例子:

```js
function Car(name,price) {
  this.name = name;
    this.price = price;
    }
    Car.prototype.showInfo = () =>{
      alert(this === window);//true
      }
      var bus = new Car('Bus', 1000);
      bus.showInfo();
    }
}
```

虽然`.showInfo()`这个函数是Car内部的方法，但是由于这个箭头函数所被定义的地方并不是在函数内部，而是在被定义在全局环境中，所以它内部的this值其实是window对象。
但是这种情况对于普通函数来说并不会出现这个问题，这正是因为刚刚我们说过的箭头函数的this值不可改变性，对于一个普通函数来说，它内部的this值是取决于调用方式的，
当以方法调用的形式来调用普通函数的时候，普通函数的this也就是调用它的对象，所以箭头函数所遇到的这种情况相对于普通函数来说就不会发生。

## 总结

在JavaScript中this关键字一直是一个很重要的问题，这次总结了这些也是之前的一些思考，如果大家有什么想法可以多多和我交流：)

