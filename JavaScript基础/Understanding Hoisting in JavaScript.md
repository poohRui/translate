# 理解JavaScript中的变量提升

> 原文 [Understanding Hoisting in JavaScript](https://designingforscale.com/understanding-hoisting-in-javascript/)

变量提升这个词不是所有人都听说过，在这篇文章里，我会解释什么是变量提升以及使用不同的例子来让大家对变量提升有更好的理解。

## JavaScript的解释器

当大家在执行JavaScript代码的时候，实际上JavaScript解释器从头到尾将代码运行了两遍。

在第一次遍历中，解释器会进行一次安全性检查和对代码性能的一些小的优化。安全性检查就是检查代码的语法是否正确，代码中是否存在对`with`和`eval`语句等（注：使用with或eval会导致代码很难被JavaScript解释器优化，甚至在严格模式下禁用with）。然后，为了让代码在执行的时候能展示出更好的性能，解释器会对代码进行优化。变量提升也发生在这次遍历过程中，这次遍历也被称为*编译过程*

真正的一行一行的执行你的代码是在第二次遍历中，在这期间会进行变量的赋值，函数的调用等操作。

## 什么是变量提升

变量提升是指JavaScript解释器将所有变量和函数声明移动到当前作用域的顶部，实际上需要牢记的是提升的仅仅是声明，赋值操作还是停留在它们原本的位置。

在JavaScript解释器第一次遍历代码的过程中完成变量提升。

## 声明变量

让我们从一个简单的例子开始，下面是一段代码：

```javascript
'use strict'
console.log(bar);//undefined
var bar = 'bar';
console.log(bar);//'bar'
```

首先，你可能会想上面的代码在第三行会抛出`ReferenceError`的错误，因为`bar`这个变量还没有被声明，但是，由于变量提升的魔法效果，这句代码并不会抛出一个`ReferenceError`的错误，而仅仅是将`bar`的值输出为`undefined`，这是因为JavaScript解释器在第一次遍历代码的过程中就已经将所有变量和函数的声明移动到当前作用域的顶部。然后第二次遍历执行这些代码。

解释器第一次遍历后的代码如下：

```javascript
'use strict'
var bar;
console.log(bar);//'undefined'
bar = 'bar';
console.log(bar);//'bar'
```

注意到变量`bar`在顶部声明但是并没有被赋值了么？这是一个微妙但却很重要的区别，这也是输出的`bar`值是`undefined`而不是抛出一个`ReferenceError`错误的原因。

## 声明函数

函数的声明同样也用了提升(注意是函数声明而不是函数表达式)，来分析下面这段样例代码：

```javascript
'use strict';
foo()
function foo(){
    console.log(bam);//'undefined'
  	var bam = 'bam';
}
console.log(bam);//ReferenceError:bam is not defined
```

在这段代码中，因为foo是一个函数声明会被提升到当前作用域的顶部，所以可以在第一行成功的调用`foo`。然后在`foo`函数中打印的变量`bam`为`undefined`就像前面的那个例子说的一样，即`bam`由于变量提升被提升至当前作用域的顶部，即`function foo()`函数中的第一行。这就意味着，`bam`在执行`console.log(bam)`之前就被调用，但是还未执行`bam = 'bam'`的赋值操作。

这里需要特别注意的是，`bam`只是提升到**当前**作用域的顶部，`bam`的声明在function的作用域内而不是在全局作用域下声明的，所以仅仅是被提升至函数作用域的顶部。

下面是解释器第一次运行后的等价代码：

```javascript
'use strict';
function foo(){
  	var bam;
    console.log(bam);//'undefined'
  	bam = 'bam';
}
foo();
console.log(bam);//ReferenceError:bam is not defined
```

注意到`foo()`怎样被移动到顶部，`bam`是怎样被声明的了吗？这意味着，当在最后一行试图`console.log(bam)`的时候在全局作用域没有找到变量`bam`就会抛出一个`ReferenceError`。

## 函数表达式

在第三个例子中，我想介绍一下如果使用函数表达式将和使用函数声明的结果完全相反，函数表达式不会进行变量提升，取而代之的是，被赋予函数表达式的变量将会提升，下面有一个例子来证明我的观点：

```javascript
'use strict';
foo();
var foo = function(){
    console.log(bam);//'undefined'
  	var bam = 'bam';
}
```

上面的代码会抛出一个`TypeError:foo is not a function`的错误，因为只有变量`foo`被提升到了顶部，在解释器第二遍运行的时候才进行将函数赋值给`foo`的操作。

下面是解释器第一次运行后的等价代码：

```javascript
'use strict';
var foo;
foo();//`foo` has not been assigned the function yet
foo = function(){
  var bam;
  console.log(bam);
  bam = 'bam';
}
```

## 哪一个有更高的优先级？

我想介绍的最后一点是*函数声明* **先于**变量被提升，来看下面的代码：

```javascript
'use strict';
console.log(typeof foo);//'function'
var foo = 'foo';
function foo(){
    var bam = 'bam';
  	console.log(bam);
}
```

在这里例子中，`typeof foo`返回`function`而不是`string`，即使函数`foo`声明在变量`foo`的后面，这就是因为*函数声明*先于*变量声明*被提升，而且在第二遍执行的过程中`foo = 'foo'`赋值操作在`typeof foo`语句之后，所以是输出`'function'`。

在第一遍运行中，解释器会将`foo()`提升至当前作用域的顶部，然后运行到`var foo = 'foo'`这行，这个时候，解释器会发现`foo`已经被声明过了没有必要再次进行声明，所以不会做任何操作并继续向下进行解析。

然后，在第二次运行中(这次是真正的在执行代码)，会先执行`typeof foo`，然后在执行赋值`foo = 'foo'`。

下面是解释器第一次运行后的等价代码：

```javascript
'use strict';
function foo(){
    var bam = 'bam';
  	console.log(bam);
}
console.log(typeof foo);//'function
foo = 'foo';
```

## ES6

下面来看看在ES6中怎么进行变量提升。

使用`let`和`const`声明的变量进行变量提升和使用`var`来声明变量进行的变量提升有所不同，不同点在于使用`let`和`const`声明的变量虽然也进行变量提升，但是，在赋值操作执行前变量都不能被访问。(注：测试例子如下)

```javascript
console.log(a);//ReferenceError: a is not defined
let a = 10;
```

原文段引用ES6文档

> The variables are created when their containing Lexical Environment is instantiated **but may not be accessed in any way until the variable’s LexicalBinding is evaluated.**

（大致意思是：在包含变量的语义环境实例化时变量就会被创建，但是不可以被访问，直到产生绑定后才可以被访问）

最终总结出，解释器将在编译的时候提供变量提升，但是如果在变量赋值前访问该变量则会抛出reference error的错误。

## 结论

我希望这个篇文章会让你理解JavaScript中变量提升，它虽然不像想象中的那么神奇和复杂，但是确实需要仔细区分在不同场景的状态，来彻底理解在这件事情在最底层是怎么运作的。



