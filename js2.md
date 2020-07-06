## this指向

首先，我们先来回顾一下执行上下文的生命周期

![1594045172718](C:\Users\NieLei\Desktop\js2\1594045172718.png)

首先，我们需要得出一个非常重要的，并且一定要牢记于心的结论，**this的指向，是在函数被调用的时候确定的。**也就是执行上下文被创建时确定的。

因此，一个函数中的this指向，可以非常灵活。比如下面的例子中，同一个函数由于调用方式的不同，this指向了不一样的对象。



```javascript
var a = 10;
var obj = {
  a: 20
}

function fn() {
  console.log(this.a);
}

fn(); // 10
fn.call(obj); // 20
```

除此之外，**在函数执行过程中，this一旦被确定，就不可更改了。**

```javascript

var a = 10;
var obj = {
  a: 20
}

function fn() {
  this = obj; // 这句话试图修改this，运行后会报错
  console.log(this.a);
}

fn();
```

### 函数中的this

在总结函数中this指向之前，我们有必要通过一些奇怪的例子，来感受一下函数中this的捉摸不定。

```javascript

// demo01
var a = 20;
function fn() {
  console.log(this.a);
}
fn();

// demo02
var a = 20;
var obj = {
  a: 10,
  c: this.a + 20,
  fn: function () {
    return this.a;
  }
}

console.log(obj.c);
console.log(obj.fn());
```

分析之前，我们直接了当抛出结论。

在一个函数上下文中，this由调用者提供，由调用函数的方式来决定。**如果调用者函数，被某一个对象所拥有，那么该函数在调用时，内部的this指向该对象。如果函数独立调用，那么该函数内部的this，则指向undefined**。但是在非严格模式中，当this指向undefined时，它会被自动指向全局对象。

但是我们需要特别注意的是demo02。在demo02中，对象obj中的c属性使用`this.a + 20`来计算。这里我们需要明确的一点是，单独的`{}`不会形成新的作用域，因此这里的`this.a`，由于并没有作用域的限制，它仍然处于全局作用域之中。所以这里的this其实是指向的window对象

那么我们修改一下demo03的代码，大家可以思考一下会发生什么变化。

 ```javascript

'use strict';
var a = 20;
function foo() {
  var a = 1;
  var obj = {
    a: 10,
    c: this.a + 20,
    fn: function () {
      return this.a;
    }
  }
  return obj.c;
}
console.log(foo());    // ？
console.log(window.foo());  // ?
 ```

再来看一些容易理解错误的例子，加深一下对调用者与是否独立运行的理解。

 ```javascript
var a = 20;
var foo = {
  a: 10,
  getA: function () {
    return this.a;
  }
}
console.log(foo.getA()); // 10

var test = foo.getA;
console.log(test());  // 20
 ```

`foo.getA()`中，getA是调用者，他不是独立调用，被对象foo所拥有，因此它的this指向了foo。而`test()`作为调用者，尽管他与foo.getA的引用相同，但是它是独立调用的，因此this指向undefined，在非严格模式，自动转向全局window。

### 使用call，apply显示指定this

JavaScript内部提供了一种机制，让我们可以自行手动设置this的指向。它们就是call与apply。所有的函数都具有这两个方法。它们除了参数略有不同之外，其功能完全一样。它们的第一个参数都为this将要指向的对象。

如下例子所示。fn并非属于对象obj的方法，但是通过call，我们将fn内部的this绑定为obj，因此就可以使用this.a访问obj的a属性了。这就是call/apply的用法。

```javascript

function fn() {
  console.log(this.a);
}
var obj = {
  a: 20
}

fn.call(obj);
```

call与apply后面的参数，都是向将要执行的函数传递参数。其中call以一个一个的形式传递，apply以数组的形式传递。这是他们唯一的不同。

```javascript

function fn(num1, num2) {
  console.log(this.a + num1 + num2);
}
var obj = {
  a: 20
}

fn.call(obj, 100, 10); // 130
fn.apply(obj, [20, 10]); // 50
```

## 闭包

**闭包是一种特殊的对象。**

**它由两部分组成。执行上下文(代号A)，以及在该执行上下文中创建的函数（代号B）。**

**当B执行时，如果访问了A中变量对象中的值，那么闭包就会产生。**

**在大多数理解中，包括许多著名的书籍，文章里都以函数B的名字代指这里生成的闭包。而在chrome中，则以执行上下文A的函数名代指闭包。**

因此我们只需要知道，一个闭包对象，由A、B共同组成，接下来我们以chrome的标准来称呼。

```javascript

// demo01
function foo() {
  var a = 20;
  var b = 30;

  function bar() {
    return a + b;
  }

  return bar;
}

var bar = foo();
bar();
```

上面的例子，首先有执行上下文foo，在foo中定义了函数bar，而通过对外返回bar的方式让bar得以执行。当bar执行时，访问了foo内部的变量a，b。因此这个时候闭包产生

上节课中我们讲到JavaScript拥有自动的垃圾回收机制，关于垃圾回收机制，有一个重要的行为，那就是，当一个值，在内存中失去引用时，垃圾回收机制会根据特殊的算法找到它，并将其回收，释放内存。

而我们知道，函数的执行上下文，在执行完毕之后，生命周期结束，那么该函数的执行上下文就会失去引用。其占用的内存空间很快就会被垃圾回收器释放。可是闭包的存在，会阻止这一过程。

先来一个简单的例子。

```javascript

var fn = null;
function foo() {
  var a = 2;
  function innnerFoo() {
    console.log(a);
  }
  fn = innnerFoo; // 将 innnerFoo的引用，赋值给全局变量中的fn
}

function bar() {
  fn(); // 此处的保留的innerFoo的引用
}

foo();
bar(); // 2
```

在上面的例子中，`foo()`执行完毕之后，按照常理，其执行环境生命周期会结束，所占内存被垃圾收集器释放。但是通过`fn = innerFoo`，函数innerFoo的引用被保留了下来，复制给了全局变量fn。这个行为，导致了foo的变量对象，也被保留了下来。于是，函数fn在函数bar内部执行时，依然可以访问这个被保留下来的变量对象。所以此刻仍然能够访问到变量a的值。

这样，我们就可以称foo为闭包。

**所以，通过闭包，我们可以在其他的执行上下文中，访问到函数的内部变量。**比如在上面的例子中，我们在函数bar的执行环境中访问到了函数foo的a变量

> 注：虽然例子中的闭包被保存在了全局变量中，但是闭包的作用域链并不会发生任何改变。在闭包中，能访问到的变量，仍然是作用域链上能够查询到的变量。

```javascript

var fn = null;
function foo() {
  var a = 2;
  function innnerFoo() {
    console.log(c); // 在这里，试图访问函数bar中的c变量，会抛出错误
    console.log(a);
  }
  fn = innnerFoo; // 将 innnerFoo的引用，赋值给全局变量中的fn
}

function bar() {
  var c = 100;
  fn(); // 此处的保留的innerFoo的引用
}

foo();
bar();
```

### 闭包的应用场景

##### 柯里化

在函数式编程中，利用闭包能够实现很多炫酷的功能，柯里化便是其中很重要的一种。

##### 模块化

模块化是闭包最强大的一个应用场景。

```javascript

(function () {
        var a = 10;
        var b = 20;

        function add(num1, num2) {
            var a1 = num1 ? num1 : a;
            var b1 = num2 ? num2 : b;

            return a1 + b1;
        }

        window.add = add;
    })();

    add(10, 20);
```



