JavaScript 有一个特性叫做隐式全局变量（implied globals），当使用一个变量名时，JavaScript 解释器将反向遍历作用域链来查找变量的声明，如果没有找到，就假定该变量是全局变量。这种特性使得我们可以在闭包里随处引用全局变量，比如 [**jQuery**](http://lib.csdn.net/base/22) 或 window。然而，这是一种不好的方式。

考虑模块的独立性和封装，对其它对象的引用应该通过参数来引入。如果模块内需要使用其它全局对象，应该将这些对象作为参数来显式引用它们，而非在模块内直接引用这些对象的名字。以 jQuery 为例，若在参数中没有输入 jQuery 对象就在模块内直接引用 $ 这个对象，是有出错的可能的。正确的方式大致应该是这样的：

```javascript
;(function (q, w) {
  // q is jQuery
  // w is window
  // 局部变量及代码
  // 返回
})(jQuery, window);
```

相比隐式全局变量，将引用的对象作为参数，使它们得以和函数内的其它局部变量区分开来。这样做还有个好处，我们可以给那些全局对象起一个别名，比如上例中的 “q”。现在看看你的代码，是否没有经过对 jQuery 的引用就到处都是”$”？

在json文件中增加一个字段记录是预报还是实况，字段名为forecast.

增加max_length字段记录最大显示条数，超过最大显示条数，就显示成下拉列表。

增加timeStep字段，表示初始化时间的步长

step统一以时间做单位。

step中的0对应60



背景透明文字不透明用backgroundcolor reba(255,255,255,0.2)

行内元素float没有用

给img加block节省一个div

Less 自动计算css

图片和文字要放在同一行的时候，图片自己有align属性可以来规定文字和图片怎么排版

如果本身就是icon使用display inline-table vertical-align:centet

解决float:left高度不能自适应的问题，在大的包裹div中添加overfloat:hidden即可

```
label_type.push(label_choice[i].innerText);
```

遍历以同一个类名命名的所有元素



arrayObject.splice(index,howmany,item1,.....,itemX)

splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目。

先icon再文字的对齐问题后面的文字用p,样式写display: inline-block;vertical-align: top;

jquery的click和on绑定click的区别

li>span是什么



```
$.ajax({
    type: 'GET',
    url: cur_site + "team/name2mail",
    dataType: "json",
    data: {'name': search_member_name},
    success: function (data) {
        console.log(data);

    }
});
```

截取数组可以用slice，字符串也是数组的一种可以用length'



生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。
元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。

static就是正常的位置



保存按钮怎么放在中间，用个padding还会自动居中

```
[type="radio"]:checked + label:after, [type="radio"].with-gap:checked + label:after {
    background-color:  rgb(244,171,35);
}
```

```
product_label[i].className='active_product_label chip';
```

在页面上绑定了以后，就是更改class的内容，页面上的元素还是绑定的，仍然会响应绑定事件

first-child到底是怎么用的

string转为json:var obj = JSON.parse(strJSON)； 

json转为string:var str = JSON.stringify(obj) 

团队产品展示的文字浮动和居中



删除文件直接用rm命令

删除文件夹用rm -rf 命令

array.join("")把数组转换成为字符串

array.reverse()将数组里的元素进行反转



css !important使用

做导航的时候可以用$(this).addClass("selected").siblings().removeClass("selected");

jquery index的用法

$('input[id^="123"]')//选取以id 123开头的input元素



data-val存在缓存？

https://segmentfault.com/q/1010000005829016?_ea=917973