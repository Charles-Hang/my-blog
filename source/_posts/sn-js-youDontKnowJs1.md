---
title: '[学习笔记]《你不知道的JavaScript（上）》核心知识点'
date: 2017-03-29 20:38:55
tags:
- 学习笔记
categories:
- js
comments: true
---
# 第一部分 作用域和闭包
## 第4章 提升
编译器会在解释JavaScript代码之前进行编译，包括变量和函数在内的所有声明都会在任何代码被执行前首先处理，即把它们在代码中出现的位置“移动”到最上面，这个过程就是提升。示例：
```js
console.log(a);
var a = a;

==>

var a;
console.log(a); // undefined
a = a; 

====

foo();
bar();
var foo = function bar() {
    // ...
}

==>

var foo;
foo(); // TypeError
bar(); // ReferenceError
foo = function() {
    var bar = ...self...
    // ...
}
```
函数声明和变量声明都会被提升，但是函数会首先被提升，然后才是变量。示例：
```js
foo();
var foo;
function foo() {
    console.log(1);
}
foo = function() {
    console.log(2);
};

==>

function foo() {
    console.log(1);
}
foo(); // 1
foo = function() {
    console.log(2);
};
```
### 提示
声明本身会被提升，而包括函数表达式的赋值在内的赋值操作并不会提升。要注意避免重复声明，特别是当普通的var声明和函数声明混合在一起的时候，会有很多危险！
## 第5章 作用域闭包
当函数可以记住并访问所在的词法作用域时，就产生了闭包。
```js
function foo() {
    var a = 2;
    function bar() {
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz(); // 2 ---闭包的效果
```
`foo()`执行后，通常整个内部作用域都被销毁，但由于`bar()`本身在使用，就阻止了回收，它拥有`foo()`内部作用域的闭包，使得该作用域能够一直存活，以供`bar()`在之后任何时间进行引用。`bar()`一直持有对该作用域的引用，这个引用就叫作闭包。
本质上无论何时何地，如果将函数（访问它们各自的词法作用域）当作第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、Ajax请求、跨窗口通信、web workers或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包！
立即执行函数创造的新的作用域，也就是创造了闭包，只是没有在其他的作用域使用。
### 模块
闭包的一个强大应用就是模块：
```js
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another.join("!"));
    }
    return{
        doSomething: doSomething,
        doAnother: doAnoter
    };
}
var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
CoolModule()只是一个函数，必须要通过调用它来创建一个模块实例。模块模式需要具备两个必要条件：
1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

一个从函数调用所返回的，只有数据属性而没有闭包函数的对象并不是真正的模块。
es6已经提供了一个稳定的模块API供我们使用。
## 附录A 动态作用域
JavaScript中的作用域就是词法作用域，词法作用域最重要的特征是它的定义过程发生在代码的书写阶段。而动态作用域是一个在运行时被动态确定的形式，它们不关心函数和作用域时如何声明以及在何处声明的，只关心它们从何处调用。这里之所以要提及动态作用域，是因为JavaScript的this机制某种程度上很像动态作用域，它是在运行时动态确定的，this关注函数如何调用。
# 第二部分 this和对象原型
### this到底是什么
this是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。当一个函数调用时，会创建一个执行上下文，这个上下文会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this就是这个上下文的其中一个属性。
## 第2章 this全面解析
### 调用位置
找到调用位置，最重要的是要分析调用栈（就是为了达到当前执行位置所调用的所有函数）。调用位置就是在当前正在执行的函数的前一个调用中。示例：
```js
function baz() {
    // 当前调用栈是： baz
    // 因此，当前调用位置是全局作用域

    console.log( "baz");
    bar(); // <-- bar的调用位置
}
function bar() {
    // 当前调用栈是baz -> bar
    // 因此，当前调用位置在baz中

    console.log("bar");
    foo(); // <-- foo的调用位置
}
function foo() {
    // 当前调用栈是baz -> bar -> foo
    // 因此，当前调用位置在bar中

    console.log("foo");
}
baz(); // <-- baz的调用位置
```
### 绑定规则
#### 默认绑定
当函数不带任何修饰的进行调用时，即独立函数调用，使用默认绑定。默认绑定下，如果函数使用严格模式，this会绑定到undefined，否则指向全局对象。
#### 隐式绑定 
这条规则是要考虑调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。
```js
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); // 2
```
把函数当作参数赋值，隐式绑定的函数会丢失绑定对象，会应用默认绑定规则。示例：
```js
function foo() {
    console.log(this.a);
}

var obj = {
    a: 2,
    foo: foo
};

var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a是全局对象的属性
bar(); // "oops, global"
```
`bar()`调用的是`foo()`函数本身。
```js
function foo() {
    console.log(this.a);
}
function doFoo(fn) {
    // fn 其实引用的是foo

    fn(); // <-- 调用位置
}
var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // a是全局对象的属性
doFoo(obj.foo); // "oops, global"
```
参数传递其实就是一种隐式赋值，结果和上一个例子一样。
#### 显示绑定
使用Function.prototype.call Function.prototype.apply Function.prototype.bind 方法可以强制将this绑定到某一上下文上。
#### new绑定
JavaScript虽然也有一个new操作符，但它与其他面向类的语言安全不同，并不存在所谓的构造函数，只有对于函数的构造调用。使用new来调用函数，会自动执行下面的操作：
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行【【原型】】连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

new是最后一种可以影响函数调用时this绑定行为的方法，称之为new绑定。
### 优先级
new绑定 > 显示绑定 > 隐式绑定 > 默认绑定
### 一种情况
把null或者undefined作为this的绑定对象传入call、apply或bind时，应用的是默认绑定规则。这时为了不对全局对象产生影响，可以用Object.create(null)创建一个空对象将this绑定在这个空对象中：`foo.apply(Object.create(null),[1,2,3])`。
## 第3章 对象
### 类型
举个例子来说明：
```js
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; //false

var strObject = new String("I am a string");
typeof strObject; // "object"
strObject instanceof String; // true

Object.prototype.toString.call(strObject); // [object String]
```
上面strPrimitive只是一个string基本类型，不可变的值，而strObject是一个String对象。之所以可以直接对string基本类型使用String对象的方法是因为JavaScript会在必要时自动把字符串字面量转换成一个String对象，数值字面量也是如此，因此定义时推荐用字面量的形式，简洁方便。
### 内容
在对象中，属性名永远都是字符串。如果你是用string（字面量）以外的其他值作为属性名，那它也会首先被转换成字符串，即使数字也不例外，虽然在数组下标中使用的是数字，但在对象属性名中数字会被转换成字符串，所以当心不要搞混对象和数值中数字的用法：
```js
var myObject = { };
myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"]; // "foo"
myObject["3"]; // "bar"
myObject["[object Object]"]; // "baz"
```
#### 可计算属性名
ES6 增加了可计算属性名：
```js
var prefix = "foo";

var myObject = {
[prefix + "bar"]:"hello",
[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```
#### 数组
数组也是对象，所以虽然每个下标都是整数，你仍然可以给数组添加属性：
```js
var myArray = [ "foo", 42, "bar" ];
myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```
可以看到虽然添加了命名属性，数组的length值并未发生变化。完全可以把数组当作一个普通的键/值对象来使用，但这不是一个好选择。
当试图给数组添加一个数字字符串的属性名时，它会变成一个数值下标，注意与对象区别：
```js
var myArray = [ "foo", 42, "bar" ];
myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```
#### 属性描述符
直接上代码：
```js
var myObject = {
    a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// value: 2,
// writable: true,
// enumerable: true,
// configurable: true
// }

var myObject = {};
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
});
myObject.a; // 2
```
writable和enumerable不用说，configurable为false时，就不能用`defineProperty()`方法来修改属性描述符，也不能用delete语句删除这个属性。这里有个小例外：即便属性是不可配置的，我们还是可以把是否可写状态由true改为false，不过无法由false改为true。
#### 不变性
对象常量
结合`writable:false`和`configurable:false`就可以创建一个真正的常量属性（不可修改、重定义或删除）：
```js
var myObject = {};
Object.defineProperty( myObject, "FAVORITE_NUMBER", {
    value: 42,
    writable: false,
    configurable: false
});
```
禁止扩展
可以使用`Object.preventExtensions()`来禁止一个对象添加新属性：
```js
var myObject = {
    a:2
};
Object.preventExtensions( myObject );
myObject.b = 3;
myObject.b; // undefined
```
严格模式下会抛出错误。
密封
`Object.seal()`会创建一个“密封”对象，相当于`Object.preventExtensions() + configurable:false`
冻结
`Object.freeze`会创建一个冻结对象，相当于`Object.seal() + writable:false`。冻结是可以用在一个对象上的级别最高的不可变性。
#### Getter和Setter（访问描述符）
直接上代码：
```js
var myObject = {
    // 给a 定义一个getter
    get a() {
        return 2;
    }
};
Object.defineProperty(
    myObject, // 目标对象
    "b", // 属性名
    { // 描述符
        // 给b 设置一个getter
        get: function(){ return this.a * 2 },
        // 确保b 会出现在对象的属性列表中
        enumerable: true
    }
);
myObject.a; // 2
myObject.b; // 4
```
setter同理，当只定义一个时，会忽略另一个操作，通常getter和setter是成对出现的，否则可能才产生意料之外的行为。
## 第4章 混合对象“类”
### 混入
#### 显示混入
```js
// 非常简单的mixin()例子：
function mixin(sourceObj, targetObj) {
    for(var key in sourceObj) {
        // 只会在不存在的情况下复制 但也只是复制引用
        if(!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}
var Vehicle = {
    engines: 1,
    ignition: function() {
        console.log("Turning on my engine.");
    },
    drive: function() {
        this.ignition();
        console.log("steering and moving forward!");
    }
};
var Car = mixin(Vehicle, {
    wheels: 4,
    drive: function() {
        Vehicle.drive.call(this);  // 显式多态
        console.log("Rolling on all " + this.wheels + " wheels!");
    }
});
```
上面看似是很棒的机制，实际上并不能带来太多好处，要注意只在能提高代码可读性的前提下使用显示混入，避免使用增加代码理解难度或者让对象关系更加复杂的模式。
##### 寄生继承
显示混入模式的一种变体，它既是显式的又是隐式的。
```js
//"传统的JavaScript类"Vehicle
function Vehicle() {
    this.engines = 1;
}
Vehicle.prototype.ignition = function() {
    console.log("Turning on my engine.");
};
vehicle.prototype.drive = function() {
    this.ignition();
    console.log("Steering and moving forward!");
};
//"寄生类"Car
function Car() {
    //首先，car是一个Vehicle
    var car = new Vehicle();
    //接着对car进行定制
    car.wheels = 4;
    //保存到Vehicle::drive()的特殊引用
    var vehDrive = car.drive;
    //重写Vehicle::drive()
    car.drive = function() {
        vehDrive.call(this);
        console.log("Rolling on all " + this.wheels + " wheels!");
    }
    return car;
}
var myCar = new Car();
myCar.drive();
```
#### 隐式混入
```js
var Something = {
    cool: function() {
        this.greeting = "Hello World";
        this.count = this.count ? this.count + 1 : 1;
    }
};
Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
    cool: function() {
        // 隐式把Something 混入Another
        Something.cool.call( this );
    }
};
Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 （count 不是共享状态）
```
这类技术虽利用了this的重新绑定功能，但不够灵活，通常来说，应尽量避免使用这样的结构，以保证代码的整洁和可维护性。
### 小结
混入模式虽可用来模拟类的复制行为，但通常会产生丑陋并且脆弱的语法，这会让代码更加难懂并且难以维护。况且JavaScript并不会像类一样创建对象的副本，只能复制引用。
## 第5章 原型
#### 属性设置和屏蔽
分析下面这个给一个对象设置属性的过程：
`myObject.foo = "bar";`
如果myObject中包含foo的普通数据访问属性，这条赋值语句只会修改已有的属性值。
如果myObject中不存在foo，原型链上也找不到foo，foo就会被直接添加到myObject上。
如果foo既出现在myObject中也出现在myObject的原型链中，就会发生屏蔽。myObject 中包含的foo 属性会屏蔽原型链上层的所有foo 属性。
但是如果foo不直接存在与myObject中而是存在原型链上时，`myObject.foo = "bar";`会出现三种情况：
1. 如果在原型链上存在名为foo 的普通数据访问属性，并且没有被标记为只读（writable:false），那就会直接在myObject 中添加一个名为foo 的新属性，它是屏蔽属性。
2. 如果在原型链上存在foo，但是它被标记为只读（writable:false），那么无法修改已有属性或者在myObject 上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在原型链上存在foo 并且它是一个setter，那就一定会调用这个setter。foo不会被添加到（或者说屏蔽于）myObject，也不会重新定义foo 这个setter。

只有第一种才会触发屏蔽，如果是第二、第三种就不能使用`=`来赋值了，就要使用`Object.defineProperty()`来向myObject添加foo.
一些情况会产生隐式屏蔽：
```js
var anotherObject = {
    a:2
};
var myObject = Object.create( anotherObject );
anotherObject.a; // 2
myObject.a; // 2
anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // 隐式屏蔽！

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```
`myObject.a++;`相当于`myObject.a = myObject.a + 1;`这样给myObject新建了屏蔽属性a！
### “类”
#### “构造函数”
```js
function Foo() {
    // ...
}
Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```
Foo.prototype默认有一个共有并且不可枚举的属性`.constructor`,这个属性引用的是对象关联的函数（本例是Foo）。a本身并没有`.constructor`属性，只是委托给了Foo.prototype，这和“构造”毫无关系。对于`.constructor`的错误理解很容易对你自己产生误导。例：
```js
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```
Object()并没有“构造”a1，看起来应该是Foo()“构造”了它，这次的结果是因为Foo.prototype也没有`.constructor`属性，继续向上委托到了Object.prototype。
当然，此时可以给Foo.prototype通过Object.defineProperty添加一个不可枚举的`.constructor`属性来达到最开始的效果。
### （原型）继承
典型的“原型风格”：
```js
function Foo(name) {
    this.name = name;
}
Foo.prototype.myName = function() {
    return this.name;
};

function Bar(name,label) {
    Foo.call( this, name );
    this.label = label;
}

// 我们创建了一个新的Bar.prototype 对象并关联到Foo.prototype
Bar.prototype = Object.create( Foo.prototype );
// 注意！现在没有Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它

Bar.prototype.myLabel = function() {
    return this.label;
};

var a = new Bar( "a", "obj a" );
a.myName(); // "a"
a.myLabel(); // "obj a"
```
上面代码的核心部分就是语句`Bar.prototype = Object.create(Foo.prototype)`。调用`Object.create()`会凭空创建一个“新”对象并把新对象内部的原型关联到你指定的对象（本例中是Foo.prototype）。ES6新增了一种方法：`Object.setPrototypeOf( Bar.prototype, Foo.prototype );`也可以实现。
#### 检查“类”关系
检查一个实例（JavaScript中的对象）的继承祖先（JavaScript中的委托关联）通常被称为内省（或者反射）。
`b.isPrototypeOf( c );`b是否出现在c的原型链中。也可以直接获取一个对象的原型链`Object.getPrototypeOf( a );`。`.__proto__`引用了对象的原型对象，在ES6中也成为了标准。
### 对象关联
```js
var anotherObject = {
    a:2
};
var myObject = Object.create( anotherObject, {
    b: {
    enumerable: false,
    writable: true,
    configurable: false,
    value: 3
    },
    c: {
    enumerable: true,
    writable: false,
    configurable: false,
    value: 4
    }
});

myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true

myObject.a; // 2
myObject.b; // 3
myObject.c; // 4
```
为了使代码更具可读性，更易维护，推荐使用内部委托：
```js
var anotherObject = {
    cool: function() {
        console.log( "cool!" );
    }
};
var myObject = Object.create( anotherObject );
myObject.doCool = function() {
    this.cool(); // 内部委托！
};
myObject.doCool(); // "cool!"
```
## 第6章 行为委托
#### 委托理论
```js
Task = {
    setID: function(ID) { this.id = ID; },
    outputID: function() { console.log( this.id ); }
};
// 让XYZ 委托Task
XYZ = Object.create( Task );
XYZ.prepareTask = function(ID,Label) {
    this.setID( ID );
    this.label = Label;
};
XYZ.outputTaskDetails = function() {
    this.outputID();
    console.log( this.label );
};
// ABC = Object.create( Task );
// ABC ... = ...
```
通常来说，在原型委托中最好把状态保存在委托者（XYZ、ABC）而不是委托目标（Task）上。尽量避免在原型链的不同级别中使用相同的命名，用以消除引用歧义。委托行为意味着某些对象（XYZ）在找不到属性或者方法引用时会把这个请求委托给另一个对象（Task）。要禁止互相委托。
#### 比较思维模型
典型的（“原型”）面向对象风格：
```js
function Foo(who) {
    this.me = who;
}
Foo.prototype.identify = function() {
    return "I am " + this.me;
};
function Bar(who) {
    Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );
Bar.prototype.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );
b1.speak();
b2.speak();
```
对象关联风格实现功能完全相同的代码：
```js
Foo = {
    init: function(who) {
        this.me = who;
    },
    identify: function() {
        return "I am " + this.me;
    }
};
Bar = Object.create( Foo );
Bar.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};
var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );
b1.speak();
b2.speak();
```
### 类与对象
ES5实现类风格的代码：
```js
// 父类
function Widget(width,height) {
    this.width = width || 50;
    this.height = height || 50;
    this.$elem = null;
}
Widget.prototype.render = function($where){
    if (this.$elem) {
        this.$elem.css( {
            width: this.width + "px",
            height: this.height + "px"
        } ).appendTo( $where );
    }
};

// 子类
function Button(width,height,label) {
    // 调用“super”构造函数
    Widget.call( this, width, height );
    this.label = label || "Default";
    this.$elem = $( "<button>" ).text( this.label );
}
// 让Button“继承”Widget
Button.prototype = Object.create( Widget.prototype );
// 重写render(..)
Button.prototype.render = function($where) {
    // “super”调用
    Widget.prototype.render.call( this, $where );
    this.$elem.click( this.onClick.bind( this ) );
};
Button.prototype.onClick = function(evt) {
    console.log( "Button '" + this.label + "' clicked!" );
};
$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = new Button( 125, 30, "Hello" );
    var btn2 = new Button( 150, 40, "World" );
    btn1.render( $body );
    btn2.render( $body );
} );
```
代码中丑陋的显式伪多态`Widget.call`和`Widget.prototype.render.call`。
使用ES6的class语法糖实现的功能相同的代码：
```js
class Widget {
    constructor(width,height) {
        this.width = width || 50;
        this.height = height || 50;
        this.$elem = null;
    }
    render($where){
        if (this.$elem) {
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px"
            } ).appendTo( $where );
        }
    }
}
class Button extends Widget {
    constructor(width,height,label) {
        super( width, height );
        this.label = label || "Default";
        this.$elem = $( "<button>" ).text( this.label );
    }
    render($where) {
        super( $where );
        this.$elem.click( this.onClick.bind( this ) );
    }
    onClick(evt) {
        console.log( "Button '" + this.label + "' clicked!" );
    }
}
$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = new Button( 125, 30, "Hello" );
    var btn2 = new Button( 150, 40, "World" );
    btn1.render( $body );
    btn2.render( $body );
} );
```
丑陋的语法都不见了，非常棒。
使用对象关联风格委托来实现功能相同的代码：
```js
var Widget = {
    init: function(width,height){
        this.width = width || 50;
        this.height = height || 50;
        this.$elem = null;
    },
    insert: function($where){
        if (this.$elem) {
            this.$elem.css( {
                width: this.width + "px",
                height: this.height + "px"
            } ).appendTo( $where );
        }
    }
};
var Button = Object.create( Widget );
Button.setup = function(width,height,label){
    // 委托调用
    this.init( width, height );
    this.label = label || "Default";
    this.$elem = $( "<button>" ).text( this.label );
};
Button.build = function($where) {
    // 委托调用
    this.insert( $where );
    this.$elem.click( this.onClick.bind( this ) );
};
Button.onClick = function(evt) {
    console.log( "Button '" + this.label + "' clicked!" );
};
$( document ).ready( function(){
    var $body = $( document.body );
    var btn1 = Object.create( Button );
    btn1.setup( 125, 30, "Hello" );
    var btn2 = Object.create( Button );
    btn2.setup( 150, 40, "World" );
    btn1.build( $body );
    btn2.build( $body );
} );
```
之前的一次调用变成了两次，需要初始化。这看似是个缺点，实则更好地支持关注分离原则，更灵活了。
### 更好地语法
ES6的class语法可以简洁地定义类方法，ES6中也可以在任意对象的字面量形式中使用简洁方法声明，唯一的区别是对象的字面量形式仍然需要使用逗号来分隔元素，而class语法不需要。不过有一个小缺点：
```js
var Foo = {
    bar() { /*..*/ },
    baz: function baz() { /*..*/ }
};
```
去掉语法糖之后的代码如下：
```js
var Foo = {
    bar: function() { /*..*/ },
    baz: function baz() { /*..*/ }
};
```
简洁形式的声明最终会变成一个匿名函数表达式，这样自我引用（递归、事件（解除）绑定、等等）更难。
### 小结
本章中我们看到了另一种更少见但是更强大的设计模式：行为委托。对象关联（对象之前互相关联）是一种编码风格，它倡导的是直接创建和关联对象，不把它们抽象成类。对象关联可以用基于原型的行为委托非常自然地实现。
## 附录A ES6中的Class
class语法很好的解决了一些典型原型风格代码中的许多显而易见的语法问题和缺点，但它毕竟不是新的类机制，只是现有原型机制的一种语法糖，自然有许多缺点没有解决或者新暴露了出来。
### class陷阱
class语法无法定义类成员属性（只能定义方法），这其实是把双刃剑，好处是原型链末端的“实例”不会意外地获取其他地方的属性（这些属性隐式被所有“实例”所共享），但缺点就是如果要跟踪“实例”之间的共享状态，且必须要这么做，就只能使用丑陋的`.prototype`语法：
```js
class c {
    constructor() {
        // 确保修改的是共享状态而不是在实例上创建一个屏蔽属性！
        c.prototype.count++;

        // this.count 可以通过委托实现我们想要的功能
        console.log("Hello: " + this.count);
    }
}
// 直接向prototype对象上添加一个共享状态
C.prototype.count = 0;

var c1 = new C(); // Hello: 1
var c2 = new C(); // Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```
这个方法最大的问题就是违背了class语法的本意，在实现中暴露了`.prototype`。
此外，class语法仍勉励意外屏蔽的问题：
```js
class C {
    constructor(id) {
        // 噢，郁闷，我们的id 属性屏蔽了id() 方法
        this.id = id;
    }
    id() {
        console.log( "Id: " + id );
    }
}

var c1 = new C( "c1" );
c1.id(); // TypeError -- c1.id 现在是字符串"c1"
```
除此之外，super也存在一些细微的问题，处于性能考虑，super并不是动态绑定的，它会在声明时“静态”绑定。
```js
class P {
    foo() { console.log( "P.foo" ); }
}
class C extends P {
    foo() {
        super();
    }
}

var c1 = new C();
c1.foo(); // "P.foo"
var D = {
    foo: function() { console.log( "D.foo" ); }
};
var E = {
    foo: C.prototype.foo
};

// 把E 委托到D
Object.setPrototypeOf( E, D );
E.foo(); // "P.foo"
```
大家可能期望super()会自动识别出E委托了D，所以E.foo()中的super()应该调用D.foo()，但实际并不是这样！这里只能手动修改super绑定：
```js
var D = {
    foo: function() { console.log( "D.foo" ); }
};
// 把E 委托到 D
var E = Object.create( D );
// 手动把foo 的[[HomeObject]] 绑定到E，E.[[Prototype]] 是D， 所以 super() 是D.foo()
E.foo = C.prototype.foo.toMethod( E, "foo" );
E.foo(); // "D.foo"
```
toMethod(..) 会复制方法并把homeObject 当作第一个参数（也就是我们传入的E），第二个参数（可选）是新方法的名称（默认是原方法名）。

****
以上总结的知识要点只是现阶段我认为对我价值最大的部分，我略去了其中相当篇幅的我不感兴趣的还有暂时对我用处不大的部分，做这个总结不是为了将原书抛掉，只是想更精炼的提纯，提纯过程中对知识的再吸收，组织化。也方便之后的复习，提取。