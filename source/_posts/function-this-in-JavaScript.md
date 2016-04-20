title: "function & this in JavaScript"
date: 2014-12-03 18:48:53
categories: 转载
tags: JavaScript
---
## 简介
javascript中的函数不同于其他的语言，每个函数都是作为一个对象被维护和运行的。通过函数对象的性质，可以很方便的将一个函数赋值给一个变量或者将函数作为参数传递。在继续讲述之前，先看一下函数的使用语法：

以下是引用片段：
```JavaScript
function func1(…){…}
var func2=function(…){…};
var func3=function func4(…){…};
var func5=new Function(); 
```

这些都是声明函数的正确语法。它们和其他语言中常见的函数或之前介绍的函数定义方式有着很大的区别。那么在JavaScript中为什么能这么写？它所遵循的语法是什么呢？下面将介绍这些内容：

<!--more-->
## 认识函数对象（Function Object）

可以用function关键字定义一个函数，并为每个函数指定一个函数名，通过函数名来进行调用。在JavaScript解释执行时，函数都是被维护为一个对象，这就是要介绍的函数对象（Function Object）。

<span style="color:red">函数对象与其他用户所定义的对象有着本质的区别，这一类对象被称之为内部对象</span>，例如日期对象（Date）、数组对象（Array）、字符串对象 （String）都属于内部对象。这些内置对象的构造器是由JavaScript本身所定义的：通过执行new Array()这样的语句返回一个对象，<span style="color:red">JavaScript内部有一套机制来初始化返回的对象，而不是由用户来指定对象的构造方式。</span>

在JavaScript中，函数对象对应的类型是Function，正如数组对象对应的类型是Array，日期对象对应的类型是Date一样，可以通过 new Function()来创建一个函数对象，也可以通过function关键字来创建一个对象。为了便于理解，我们比较函数对象的创建和数组对象的创建。先 看数组对象：下面两行代码都是创建一个数组对象myArray：

以下是引用片段：
```JavaScript
var myArray=[];
//等价于
var myArray=new Array();
```
同样，下面的两段代码也都是创建一个函数myFunction：
```JavaScript
function myFunction(a,b){
    return a+b;
}
//等价于
var myFunction=new Function("a","b","return a+b"); 
```

通过和构造数组对象语句的比较，可以清楚的看到函数对象本质，前面介绍的函数声明是上述代码的第一种方式，而<span style="color:red">在解释器内部，当遇到这种语法时，就会自动构 造一个Function对象，将函数作为一个内部的对象来存储和运行。从这里也可以看到，一个函数对象名称（函数变量）和一个普通变量名称具有同样的规 范，都可以通过变量名来引用这个变量，但是函数变量名后面可以跟上括号和参数列表来进行函数调用。</sapn>

用new Function()的形式来创建一个函数不常见，因为一个函数体通常会有多条语句，如果将它们以一个字符串的形式作为参数传递，代码的可读性差。下面介绍一下其使用语法：

以下是引用片段：
```JavaScript
var funcName=new Function(p1,p2,...,pn,body);
```

参数的类型都是字符串，p1到pn表示所创建函数的参数名称列表，body表示所创建函数的函数体语句，funcName就是所创建函数的名称。可以不指定任何参数创建一个空函数，不指定funcName创建一个无名函数，当然那样的函数没有任何意义。

需要注意的是，p1到pn是参数名称的列表，即p1不仅能代表一个参数，它也可以是一个逗号隔开的参数列表，例如下面的定义是等价的：

以下是引用片段：
```JavaScript
new  Function("a", "b", "c", "return a+b+c")
new Function("a, b, c", "return a+b+c")
new Function("a,b", "c", "return a+b+c")
```
JavaScript引入Function类型并提供new Function()这样的语法是因为函数对象添加属性和方法就必须借助于Function这个类型。

函数的本质是一个内部对象，由JavaScript解释器决定其运行方式。通过上述代码创建的函数，在程序中可以使用函数名进行调用。本节开头列出的函数定义问题也得到了解释。<span style="color:red">注意可直接在函数声明后面加上括号就表示创建完成后立即进行函数调用</span>，例如：

以下是引用片段：
```JavaScript
var i=function (a,b){
return a+b;
}(1,2);
alert(i); 
```

这段代码会显示变量i的值等于3。i是表示返回的值，而不是创建的函数，因为<span style="color:red">括号“(”比等号“=”有更高的优先级。</span>这样的代码可能并不常用，但当用户想在很长的代码段中进行模块化设计或者想避免命名冲突，这是一个不错的解决办法。


需要注意的是，尽管下面两种创建函数的方法是等价的：

以下是引用片段：
```JavaScript
function funcName(){
//函数体
}
//等价于
var funcName=function(){
//函数体
} 
```

<span style="color:red">但前面一种方式创建的是有名函数，而后面是创建了一个无名函数，只是让一个变量指向了这个无名函数。在使用上仅有一点区别，就是：对于有名函数，它可以出现在调用之后再定义；而对于无名函数，它必须是在调用之前就已经定义。</span> 
例如：

以下是引用片段：
```JavaScript
<script language="JavaScript" type="text/javascript">
<!--
func();
var func=function(){
alert(1)
}
//-->
</script> 
```

这段语句将产生func未定义的错误，而：

以下是引用片段：
```JavaScript
<script language="JavaScript" type="text/javascript">
<!--
func();
function func(){
alert(1)
}
//-->
</script> 
```
则能够正确执行，下面的语句也能正确执行：

以下是引用片段：
```JavaScript
<script language="JavaScript" type="text/javascript">
<!--
func();
var someFunc=function func(){
alert(1)
}
//-->
</script> 
```
<span style="color:red">由此可见，尽管JavaScript是一门解释型的语言，但它会在函数调用时，检查整个代码中是否存在相应的函数定义，这个函数名只有是通过function funcName()形式定义的才会有效，而不能是匿名函数。</span>


## 函数对象和其他内部对象的关系


除了函数对象，还有很多内部对象，比如：Object、Array、Date、RegExp、Math、Error。这些名称实际上表示一个类型，可以通 过new操作符返回一个对象。然而函数对象和其他对象不同，当用typeof得到一个函数对象的类型时，它仍然会返回字符串“function”，而 typeof一个数组对象或其他的对象时，它会返回字符串“object”。

下面的代码示例了typeof不同类型的情况：

以下是引用片段：

```JavaScript
alert(typeof(Function));
alert(typeof(new Function()));
alert(typeof(Array));
alert(typeof(Object));
alert(typeof(new Array()));
alert(typeof(new Date()));
alert(typeof(new Object())); 
```

运行这段代码可以发现：前面4条语句都会显示“function”，而后面3条语句则显示“object”，可见new一个function实际上是返回 一个函数。这与其他的对象有很大的不同。其他的类型Array、Object等都会通过new操作符返回一个普通对象。  (**可以在控制台中进行测试**)
尽管函数本身也是一个对象，但它与 普通的对象还是有区别的，因为它同时也是对象构造器，也就是说，可以new一个函数来返回一个对象，这在前面已经介绍。所有typeof返回 “function”的对象都是函数对象。也称这样的对象为构造器（constructor），因而，所有的构造器都是对象，但不是所有的对象都是构造 器。

<span style="color:red">既然函数本身也是一个对象，它们的类型是function，联想到C++、Java等面向对象语言的类定义，可以猜测到Function类型的作用所在， 那就是可以给函数对象本身定义一些方法和属性，借助于函数的prototype对象，可以很方便地修改和扩充Function类型的定义。</span>

例如下面扩展了 函数类型Function，为其增加了method1方法，作用是弹出对话框显示"function"：

以下是引用片段：
```JavaScript
Function.prototype.method1=function(){
    alert("function");
}
function func1(a,b,c){
    return a+b+c;
}
func1.method1();
func1.method1.method1(); 
```

注意最后一个语句：func1.method1.mehotd1()，它调用了method1这个函数对象的method1方法。虽然看上去有点容易混 淆，但仔细观察一下语法还是很明确的：这是一个递归的定义。因为method1本身也是一个函数，所以它同样具有函数对象的属性和方法，所有对 Function类型的方法扩充都具有这样的递归性质。

Function是所有函数对象的基础，而Object则是所有对象（包括函数对象）的基础。在JavaScript中，任何一个对象都是 Object的实例，因此，可以修改Object这个类型来让所有的对象具有一些通用的属性和方法，修改Object类型是通过prototype来完成 的：

以下是引用片段：
```JavaScript
Object.prototype.getType=function(){
return typeof(this);
}
var array1=new Array();
function func1(a,b){
return a+b;
}
alert(array1.getType());  //object
alert(func1.getType());   //function
```

上面的代码为所有的对象添加了getType方法，作用是返回该对象的类型。两条alert语句分别会显示“object”和“function”。


## 将函数作为参数传递

<span style="color:red">在前面已经介绍了函数对象本质，每个函数都被表示为一个特殊的对象，可以方便的将其赋值给一个变量，再通过这个变量名进行函数调用。作为一个变量，它可以 以参数的形式传递给另一个函数。</span>
这在前面介绍JavaScript事件处理机制中已经看到过这样的用法，例如下面的程序将func1作为参数传递给 func2：

以下是引用片段：
```JavaScript
function func1(theFunc){
theFunc();
}
function func2(){
alert("ok");
}
func1(func2); 
```

在最后一条语句中，func2作为一个对象传递给了func1的形参theFunc，再由func1内部进行theFunc的调用。<span style="color:red">事实上，将函数作为参数传递，或者是将函数赋值给其他变量是所有事件机制的基础。</span>


例如，如果需要在页面载入时进行一些初始化工作，可以先定义一个init的初始化函数，再通过window.onload=init;语句将其绑定到页面载入完成的事件。这里的init就是一个函数对象，它可以加入window的onload事件列表。


## 传递给函数的隐含参数：arguments

当进行函数调用时，除了指定的参数外，还创建一个隐含的对象——arguments。arguments是一个类似数组但不是数组的对象，说它类似是因为 它具有数组一样的访问性质，可以用arguments[index]这样的语法取值，拥有数组长度属性length。arguments对象存储的是实际传递给函数的参数，而不局限于函数声明所定义的参数列表，例如：

以下是引用片段：
```JavaScript
function func(a,b){
    alert(a);
    alert(b);
    for(var i=0;i<arguments.length;i++){
         alert(arguments[i]);
    }
}
func(1,2,3); 
```

代码运行时会依次显示：1，2，1，2，3。因此，在定义函数的时候，即使不指定参数列表，仍然可以通过arguments引用到所获得的参数，这给编程 带来了很大的灵活性。<span style="color:red">arguments对象的另一个属性是callee，它表示对函数对象本身的引用，这有利于实现无名函数的递归或者保证函数的封装性。</span>

例如使用递归来计算1到n的自然数之和：

以下是引用片段：
```JavaScript
var sum=function(n){
    if(1==n) {
        return 1;
    }
    else {
       return n+sum(n-1);
    }
}
alert(sum(100)); 
```

其中函数内部包含了对sum自身的调用，然而对于JavaScript来说，函数名仅仅是一个变量名，在函数内部调用sum即相当于调用一个全局变量，不能很好的体现出是调用自身，所以使用arguments.callee属性会是一个较好的办法：

以下是引用片段：
```JavaScript
var sum=function(n){
    if(1==n) {
        return 1;
    }
    else {
        return n+arguments.callee(n-1);
    } 
}
alert(sum(100)); 
```

<span style="color:red">callee属性并不是arguments不同于数组对象的惟一特征</span>

下面的代码说明了arguments不是由Array类型创建：

以下是引用片段：
```JavaScript
Array.prototype.p1=1;
alert(new Array().p1);
function func(){
    alert(arguments.p1);
}
func();
```
运行代码可以发现，第一个alert语句显示为1，即表示数组对象拥有属性p1，而func调用则显示为“undefined”，即p1不是arguments的属性，由此可见，<span style="color:red">arguments并不是一个数组对象。</span>

## 函数的apply、call方法和length属性

<span style="color:red">JavaScript为函数对象定义了两个方法：apply和call，它们的作用都是将函数绑定到另外一个对象上去运行</span>，两者仅在定义参数的方式有所区别：

以下是引用片段：
```JavaScript
Function.prototype.apply(thisArg,argArray);
Function.prototype.call(thisArg[,arg1[,arg2…]]); 
```

从函数原型可以看到，<span style="color:red">第一个参数都被取名为thisArg，即所有函数内部的this指针都会被赋值为thisArg，这就实现了将函数作为另外一个对象 的方法运行的目的。两个方法除了thisArg参数，都是为Function对象传递的参数。</span>

下面的代码说明了apply和call方法的工作方式：

以下是引用片段：
```JavaScript
//定义一个函数func1，具有属性p和方法A
function func1(){
    this.p="func1-";
    this.A=function(arg){
        alert("A: "+this.p+arg);
    }
}
//定义一个函数func2，具有属性p和方法B
function func2(){
    this.p="func2-";
    this.B=function(arg){
        alert("B: "+this.p+arg);
    }
}

//Function.prototype.apply(thisArg,argArray);
//Function.prototype.call(thisArg[,arg1[,arg2…]]); 

var obj1=new func1();
var obj2=new func2();
obj1.A("byA");    //显示 A: func1-byA
obj2.B("byB");    //显示 B: func2-byB
obj1.A.apply(obj2,["byA"]); //显示 A: func2-byA，其中[“byA”]是仅有一个元素的数组，下同
obj2.B.apply(obj1,["byB"]); //显示 B: func1-byB
obj1.A.call(obj2,"byA");  //显示   A: func2-byA
obj2.B.call(obj1,"byB");  //显示   B: func1-byB 
```
可以看出，obj1的方法A被绑定到obj2运行后，整个函数A的运行环境就转移到了obj2，即this指针指向了obj2。同样obj2的函数B也可以绑定到obj1对象去运行。代码的最后4行显示了apply和call函数参数形式的区别。


<span style="color:red">与arguments的length属性不同，函数对象还有一个属性length，它表示函数定义时所指定参数的个数，而非调用时实际传递的参数个数。</span>

例如下面的代码将显示2：

以下是引用片段：
```JavaScript
function sum(a,b){
    return a+b;
}
alert(sum.length);  
```

## 深入认识JavaScript中的this指针


this指针是面向对象程序设计中的一项重要概念，它表示当前运行的对象。在实现对象的方法时，可以使用this指针来获得该对象自身的引用。

<span style="color:red">和其他面向对象的语言不同，JavaScript中的this指针是一个动态的变量，一个方法内的this指针并不是始终指向定义该方法的对象的</span>，在上一节讲函数的apply和call方法时已经有过这样的例子。
为了方便理解，再来看下面的例子：

以下是引用片段：
```JavaScript
<script language="JavaScript" type="text/javascript">
<!--
//创建两个空对象
var obj1=new Object();
var obj2=new Object();
//给两个对象都添加属性p，并分别等于1和2
obj1.p=1;
obj2.p=2;
//给obj1添加方法，用于显示p的值
obj1.getP=function(){
alert(this.p); //表面上this指针指向的是obj1
}
//调用obj1的getP方法
obj1.getP();
//使obj2的getP方法等于obj1的getP方法
obj2.getP=obj1.getP;
//调用obj2的getP方法
obj2.getP();
//-->
</script> 
```
从代码的执行结果看，分别弹出对话框显示1和2。由此可见，getP函数仅定义了一次，在不同的场合运行，显示了不同的运行结果，这是有this指针的变 化所决定的。在obj1的getP方法中，this就指向了obj1对象，而在obj2的getP方法中，this就指向了obj2对象，并通过 this指针引用到了两个对象都具有的属性p。

由此可见，JavaScript中的this指针是一个动态变化的变量，它表明了当前运行该函数的对象。由this指针的性质，也可以更好的理解 JavaScript中对象的本质：<span style="color:red">一个对象就是由一个或多个属性（方法）组成的集合。每个集合元素不是仅能属于一个集合，而是可以动态的属于多个集合。 这样，一个方法（集合元素）由谁调用，this指针就指向谁。实际上，前面介绍的apply方法和call方法都是通过强制改变this指针的值来实现 的，使this指针指向参数所指定的对象，从而达到将一个对象的方法作为另一个对象的方法运行。</span>

每个对象集合的元素（即属性或方法）也是一个独立的部分，全局函数和作为一个对象方法定义的函数之间没有任何区别，因为可以把全局函数和变量看作为 window对象的方法和属性。也可以使用new操作符来操作一个对象的方法来返回一个对象，这样一个对象的方法也就可以定义为类的形式，其中的this 指针则会指向新创建的对象。在后面可以看到，这时对象名可以起到一个命名空间的作用，这是使用JavaScript进行面向对象程序设计的一个技巧。例 如：

以下是引用片段：
```JavaScript
var namespace1=new Object();
namespace1.class1=function(){
//初始化对象的代码
}
var obj1=new namespace1.class1(); 
```

这里就可以把namespace1看成一个命名空间。

由于对象属性（方法）的动态变化特性，一个对象的两个属性（方法）之间的互相引用，必须要通过this指针，而其他语言中，this关键字是可以省略的。
如上面的例子中：

以下是引用片段：
```JavaScript
obj1.getP=function(){
alert(this.p); //表面上this指针指向的是obj1
```

转载自：[http://www.cnblogs.com/yuzhongwusan/archive/2012/04/09/2438569.html](http://www.cnblogs.com/yuzhongwusan/archive/2012/04/09/2438569.html)