# 理解JavaScript中的作用域

> 原文 [Understanding Scope in JavaScript](https://developer.telerik.com/topics/web-development/understanding-scope-in-javascript/)

在JavaScript中作用域是一个非常重要但是比较模糊的概念。如果使用恰当可以成为一种痕好的设计模式来帮助你避免副作用。为了写出更好的代码，在这篇文章中，我们将分析JavaScript中不同类型的作用域和它们在底层是怎样工作的。

对作用域的一个简单的定义是*当需要调用变量或函数时，编译器去查找变量和函数的地方*，听起来是不是很简单？下面让我们来看它的全部内容。

## JavaScript解释器

在解释什么是作用域之前，我们需要讨论一下JavaScript解释器和它在不同作用域下的表现。当你在执行JavaScript代码时，实际上解释器会两次遍历你的代码。

> 更多关于变量提升的详情，请看[上一篇文章](https://designingforscale.com/understanding-hoisting-in-javascript/)

第一次遍历代码，也叫编译过程，是对作用域影响最大的一次遍历。解释器遍历代码寻找变量和函数的声明，并把它们移动到*当前作用域*的顶部。注意这里只是移动*声明*，赋值操作还是停留在它们原始的位置，在第二次遍历代码的时候，也就是执行阶段，才进行赋值操作。

为了更好的理解这个，下面有一小段简单的例子：

```javascript
'use strict';
var foo = 'foo';
var wow = 'wow';

function bar(wow){
    var pow = 'pow';
  	console.log(foo);//'foo'
  	console.log(wow);//'zoom'
}
bar('zoom');
console.log(pow);//ReferenceError:pow is not defined
```

编译过程后的等价代码如下：

```javascript
'use strict';

//函数提升
function bar(wow){
    var pow;
  	pow = 'pow';
  	console.log(foo);
  	console.log(wow);
}

//变量提升
var foo;
var wow;

foo = 'foo';
wow = 'wow';

bar('zoom');
console.log(pow);
```

理解变量和函数的声明会被提升至*当前作用域*的顶部对文章后面解释的JavaScript的作用域是至关重要的。

例如，在上面的例子中，变量`pow`在函数`bar`中声明而不是在全局变量中声明是因为`pow`的当前作用域是函数`bar`。

函数`bar`的参数`wow`也在函数作用域中被声明，实际上，所有的函数参数毫无疑问的都在函数作用域中被声明，这也就是为什么在第九行`console.log(wow)`输出的是`zoom`而不是`wow`。

## 词法作用域

到目前为止，我们介绍了JavaScript解释器是怎样工作的，也对变量提升做了一个简短的介绍，现在可以更深入的探究什么是作用域。让我们从词法作用域开始，词法作用域也被称为是编译时作用域，换句换来说，**作用域是在编译过程中就被确定的**。在这篇文章中，我们会忽略考虑代码中出现`eval`或者`with`时作用域不符合以上规律的情况，因为作为不好的编码方式，应该尽量避免出现这种特殊情况。

在解释器的第二次遍历过程中，变量被赋值，函数被调用。在上面的样例代码中，12行调用`bar()`函数就发生在这一阶段，解释器需要在调用`bar()`函数之前，从当前作用域开始一层一层向外查找`bar`函数的定义，在这个时候，当前作用域是*全局作用域*。多亏了第一次的遍历，我们知道`bar`被声明在文件的顶部，这样解释器就找到了`bar`并成功的调用它。

当我们运行到第8行的`console.log(foo)`，解释器会在执行这行之前去查找`foo`的定义。当然它做的第一件事是在当前作用域查找，即函数`bar`的作用域中（注意，当前作用域并不是全局作用域），`foo`有没有在函数作用域声明呀？并没有！然后它会向上到函数作用域的父作用域中去寻找`foo`的声明，函数的父作用域就是*全局作用域*了。`foo`用没有在全局作用域中声明呀？找到了！这样解释器就可以这句代码了。

> 总而言之，词法作用域是在第一次运行后被定义的作用域，当解释器需要寻找一个变量或者函数的声明时，它首先会在当前作用域中寻找，如果在当前作用域中找不到它需要的声明，解释器就会继续向上在上一层作用域中寻找，最终到*全局作用域*为止。

如果在*全局作用域*中都没有找到需要的声明，则会抛出一个`ReferenceError`的错误。

因为解释器总是先在当前作用域中寻找声明，如果找不到才会到父作用域中寻找，这就引出了JavaScript中的变量覆盖这一个概念，即在当前作用域中声明的变量`foo`会屏蔽父作用域中声明的同名变量，下面是一个简单的例子来帮助理解：

```javascript
'use strict'

var foo = 'foo';

function bar () {
  var foo = 'bar';
  console.log(foo);
}

bar();
```

在这个例子中，会输出*bar*而不是*foo*，因为第6行关于`foo`的声明会屏蔽第3行的声明。

变量覆盖是一种设计模式，可以保护一些变量使它们不被特定的作用域访问到。但是我个人认为最好避免这种情况发生，因为使用相同的名字来创建变量会给团队合作带来不必要的麻烦。

## 函数作用域

就像我们在词法作用域中看到的那样，解释器在当前作用域中声明变量，这就意味着，函数中的变量将在*函数作用域*中被声明。函数作用域包括函数自身和在该函数中声明的函数。

在函数作用域中声明的变量在函数外是不能被访问到的，这是一个非常有用的模式，可以用它来创建只在函数作用域中能访问到的私有属性，下面是一个例子：

```javascript
'use strict';
function convert(amount){
    var _coversionRate = 2;//只能在convert中被访问
  	return amount*_coversionRate;
}
console.log(convert(5));//10
console.log(_conversionRate);//ReferenceError:_conversionRate is not defined
```

## 块作用域

块作用域类似于函数作用域，唯一区别是块作用域是将变量局限于块而不是函数。

在ES3中，在try/catch语句中的catch语句就是一个块作用域，也就是说catch语句有自己的作用域。但是try语句却没有块级作用域，只有catch作用域有。为了能更好的理解，下面是一个例子：

```javascript
'use strict';
try{
    var foo = 'foo';
  	console.log(bar);
}
catch(err){
    console.log('In catch block');
  	console.log(err);
}
console.log(foo);
console.log(err);
```

在上面的代码中，第四行试图输出`bar`的时候会抛出错误，使解释器运行到*catch*语句中，这时候会在*catch*的作用域中声明一个`err`变量，并且这个变量不能在`catch`语句之外被访问到。所以，当我们试图在最后一行`console.log(err)`输出`err`的时候会抛出一个错误。最后实际的输出是：

```
In catch block
ReferenceError:bar is not defined
(...Error stack here...)
foo
ReferenceError:err is not defined
(...Error stack here...)
```

> 注意到变量`foo`可以在try/catch外被访问到，但是`err`不可以。

在ES6中，使用`let`和`const`声明变量，表示变量声明在块级作用域中而不是在函数作用域中，即这些变量将只能在它们被声明的块中被访问到，这个块级作用域可以是`if`块也可以是`for`块或者是一个函数。下面是一个例子：

```javascript
'use strict'
let condition = true;

function bar(){
    if(condition){
        var firstName = 'John';//Accessible in the whole function
      	let lastName = 'Doe';//Accessible in the  if only
      	const fooName = firstName + ' '+lastName;//Accessible in the  if only
    }
  	console.log(firstName);//'John'
  	console.log(lastName);//ReferenceError
  	console.log(fullName);//ReferenceError
}
bar();
```

`let`和`const`遵循的是暴露最少的原则，就是说变量应该尽可能的在范围最小的作用域中被访问到。在ES6之前，开发者通常是在IIFE中通过声明var来实现这一个原则。现在使用ES6的let和const可以很容易的做到这一点。这一原则主要优点在于避免不正确的访问变量来减少bug的可能性，并且能使垃圾回收机制能在我们离开块级作用域的第一时间就清除这些我们不用的变量。

## IIFE

立即执行函数表达式（IIFE）是一个非常流行的JavaScript的模式，允许函数创建一个新的*块级作用域*。IIFE实际上就是一个在声明完就立即执行的简单的*函数表达式*，下面是一个关于IIFE的简单例子：

```javascript
'use strict';

var foo = 'foo';
(function bar(){
    console.log('in function bar');
})()
console.log(foo);
```

上面这段代码会在输出`foo`之前先输出`in function bar`，因为函数`bar`不需要使用`bar()`这样的语句就被立即执行了，下面是造成这种情况的原因：

* `function`前面的括号使得*函数声明*变成了*函数表达式*
* 末尾的一对括号使得*函数表达式*立即被执行。

就像我们之前说的，这么做可以使得变量隐藏在当前作用域总不能被外层作用域访问到，这样可以避免对外层作用域的污染。

IIFE在执行异步操作的过程中保存变量状态非常有用，下面是这样一个例子：

```javascript
'use strict';
for(var i = 0;i<5;i++){
    setTimeout(function(){
        console.log('index:'+i);
    },1000);
}
```

我们期待的输出是0，1，2，3，4，但是由于`for`循环执行的是一个异步操作（setTimeout）实际的输出是：

```
index:5
index:5
index:5
index:5
index:5
```

出现这种情况的原因是在要执行`console.log('index:'+i)`操作的时候`for`循环已经结束了，这时候`i`的实际值是5。

如果想要输出0，1，2，3，4，就需要使用IIFE来保留我们想要的变量作用域，像下面一样：

```javascript
'use strict';

for(var i = 0;i<5;i++){
    (function logIndex(index){
        setTimeout(function(){
            console.log('index:'+index);
        },1000);
    })(i)
}
```

在上面这个例子中，我们向`IIFE`中传递了变量`i`，这样就制造了一个私有的不会受到`for`循环影响的作用域，最终输出的就是：

```
index:0
index:1
index:2
index:3
index:4
```

## 结论

JavaScript作用域有很多可以讨论的地方，但是我感觉这篇文章是对作用域的一个基础介绍，包括不同类型的作用域以及怎么样利用作用域来完成一些很好地设计模式。

我希望这篇关于作用域的接单介绍能对你有帮助。



