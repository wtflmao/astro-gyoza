---
title: Bot in Discord with discord.js (2)
date: 2022-07-09
tags: [Discord bot, JS]
category: Discord.js
summary: 这是一份非常详细和系统化的利用 NodeJS 进行 Discord bot 开发的教程，本文仅简单介绍了些 Javascript 的简单操作，非 JS 初学者可无忧跳过。
---

## Chapter 3 - JavaScript 介绍

### JS 一窥

**以下内容来自 runoob.com。感谢。侵删。**

**数组（Array）字面量** 定义一个数组：

[40, 100, 1, 5, 25, 10]

**对象（Object）字面量** 定义一个对象：

{firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"}

**函数（Function）字面量** 定义一个函数：

function myFunction(a, b) { return a * b;}


JavaScript 使用关键字 **var** 来定义变量， 使用等号来为变量赋值：

```js
var x, length  
x = 5  
length = 6
```

JavaScript 语句用于向浏览器发出命令。

语句是用分号分隔：

```js
x = 5 + 6;  
y = x * 10;
```

双斜杠 **//** 后的内容将会被忽略。

`/*` 和 `*/` 所包裹起来的代码块会被忽略。

JavaScript 语句可以写在函数内，函数可以重复引用：

**引用一个函数** = 调用函数(执行函数内的语句)。

```js
function myFunction(a, b) {  
    return a * b;                                // 返回 a 乘以 b 的结果  
}
```

JavaScript 中，常见的是驼峰法的命名规则，如 lastName (而不是lastname)。

JavaScript 语句通常以一个 **语句标识符** 为开始，并执行该语句。

语句标识符是保留关键字不能作为变量名使用。

下面列出了 JavaScript 语句标识符 (关键字) ：

- break：用于跳出循环。
- catch：语句块，在 try 语句块执行出错时执行 catch 语句块。
- continue：跳过循环中的一个迭代。
- do ... while：执行一个语句块，在条件语句为 true 时继续执行该语句块。
- for：在条件语句为 true 时，可以将代码块执行指定的次数。
- for ... in：用于遍历数组或者对象的属性（对数组或者对象的属性进行循环操作）。
- function：定义一个函数
- if ... else：用于基于不同的条件来执行不同的动作。
- return：退出函数
- switch：用于基于不同的条件来执行不同的动作。
- throw：抛出（生成）错误 。
- try：实现错误处理，与 catch 一同使用。
- var：声明一个变量。
- while：当条件语句为 true 时，执行语句块。

JavaScript 拥有动态类型。这意味着相同的变量可用作不同的类型：

```js
var x;               // x 为 undefined  
var x = 5;           // 现在 x 为数字  
var x = "John";      // 现在 x 为字符串

typeof "John"                // 返回 string  
typeof 3.14                  // 返回 number  
typeof false                 // 返回 boolean  
typeof [1,2,3,4]             // 返回 object  
typeof {name:'John', age:34} // 返回 object
```

JavaScript 对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：

```js
var person = {
    firstname:"John",
    lastname:"Doe",
    id:5566
};
```

上面例子中的对象 (person) 有三个属性：firstname、lastname 以及 id。

空格和折行无关紧要。声明可横跨多行：

```js
var person={  
firstname : "John",  
lastname  : "Doe",  
id        :  5566  
};
```

对象属性有两种寻址方式：

```js
name=person.lastname;  
name=person["lastname"];
```

下面的代码创建名为 cars 的数组：

```js
var cars=new Array();  
cars[0]="Saab";  
cars[1]="Volvo";  
cars[2]="BMW";
```

或者 (condensed array):

```js
var cars=new Array("Saab","Volvo","BMW");
```

undefined 这个值表示变量不含有值。

可以通过将变量的值设置为 null 来清空变量。

比较运算符在逻辑语句中使用，以测定变量或值是否相等。

假设 **x 等于 5**：

x == 8：_false_；
x == 5： _true_；
x == "5" ：_true_；
x === "5"：_false_；
x === 5：_true_；
x != 8：_true_；
x != "5"：_false_；
x !== 5：_false_；
x !== "5"：_true_；
x > 8：_false_；
x < 8：_true_；
x >= 8：_false_；
x <= 8：_true_；
x == undefined：_false_；
x == null：_false_；

假设 y 等于 undefined：

y == undefined：_true_
y == null：_true_
y === null：_false_

假设 z 等于 null：

z == null：_true_
z == undefined：_true_
z === undefined：_false_

逻辑与：&&
逻辑或：||
逻辑非：！

### async 和 await

下面的内容来自 CSDN 用户 loftyamb，原文链接 https://blog.csdn.net/weixin_42042017/article/details/109472908 ，感谢。

希望在一个函数执行完毕之后才去执行另外一个函数，如何解决？用 async 和 await。

await 只能在async函数中执行，否则会报错。

### 关于await

-   `await` 的意思就是**等待**。它后面可以跟一个表达式。如果是值(如字符串、数字、普通对象等等)的话，返回值就是本身的值。
-   通常使用方式为在后面跟一个 `promise` 对象。`await` 会等待这个 `promise` 的状态由 `pending` 转为 `fulfilled` 或者 `rejected`。在此期间它会阻塞，延迟执行await语句后面的语句。
-   如果 `promise` 对象的结果是 `resolve`，它会将 `resolve` 的值，作为 `await` 表达式的运算结果。

#### 例子

对于一个操作（创建菜单），需要在获取到 token 之后利用 token 才能执行这个操作，那么对应代码如下：

```js
var token = undefined;
```

```js
// 获取token
const getToken = async () => {
  global.token=await new Promise((resolve) => {
    setTimeout(() => {
      resolve("loftyamb");
    }, 3000); // 模拟通过 post 请求获取 token 返回token的过程
  });
  console.log(`成功获取到了token${global.token}`);
  // 上面的打印语句会被promise对象所阻塞，直到promise对象的状态发送改变
};
```

```js
// 利用获取到的token，进行创建菜单的操作
const createMenu = async () => {
  if (token !== undefined) {
    console.log(`利用token${token}成功创建了菜单`);
  } else {
    console.log("创建失败");
  }
};
```

那么可用以下方式实现二者的依次执行：
```js
const setMenu=async ()=>{
    await getToken();
    await createMenu();
}
```

执行：
```js
const setMenu=async ()=>{
    await getToken();
    await createMenu(); //该方法会等待前面的方法执行完毕之后才执行
}
setMenu();
```

输出结果：
```
成功获取到了tokenloftyamb
利用tokenloftyamb成功创建了菜单
```

### 关于异常

假定在获取 token 的时候发生了异常，即 Promise 对象状态为 reject，此时要在 async 函数中对异常进行处理，如下所示：

```js
// 获取token
const getToken = async () => {
  try {
    token = await new Promise((resolve, reject) => {
      setTimeout(() => {
        reject("error");
      }, 3000); // 模拟过了 3 秒才获取 token 失败
    });
  } catch (err) {
    console.log(err);
  }
  console.log(`成功获取到了token${token}`); // 异常被捕获后则此语句仍能正常执行
};
```

当异常被处理之后，则await之后的函数仍然会正常执行。

## Learn Javascript in Y minutes

原文地址：https://learnxinyminutes.com/docs/zh-cn/javascript-cn

该内容遵循 CC BY-SA 3.0

Javascript 于 1995 年由网景公司的 Brendan Eich 发明。最初它作为一种简单的，用于开发网站的脚本语言而被发明出来，是用于开发复杂网站的 Java 的补充。但由于它与网页结合度很高并且在浏览器中得到内置的支持，所以在网页前端领域 Javascript 变得比 Java 更流行了。

不过，Javascript 不仅用于网页浏览器，一个名为 Node.js 的项目提供了面向 Google Chrome V8 引擎的独立运行时环境，它正在变得越来越流行。
```js
// 注释方式和C很像，这是单行注释
/* 这是多行
   注释 */

// 语句可以以分号结束
doStuff();

// ... 但是分号也可以省略，每当遇到一个新行时，分号会自动插入（除了一些特殊情况）。
doStuff()

// 因为这些特殊情况会导致意外的结果，所以我们在这里保留分号。

///////////////////////////////////
// 1. 数字、字符串与操作符

// Javascript 只有一种数字类型(即 64位 IEEE 754 双精度浮点 double)。
// double 有 52 位表示尾数，足以精确存储大到 9✕10¹⁵ 的整数。
3; // = 3
1.5; // = 1.5

// 所有基本的算数运算都如你预期。
1 + 1; // = 2
0.1 + 0.2; // = 0.30000000000000004
8 - 1; // = 7
10 * 2; // = 20
35 / 5; // = 7

// 包括无法整除的除法。
5 / 2; // = 2.5

// 位运算也和其他语言一样；当你对浮点数进行位运算时，
// 浮点数会转换为*至多* 32 位的无符号整数。
1 << 2; // = 4

// 括号可以决定优先级。
(1 + 3) * 2; // = 8

// 有三种非数字的数字类型
Infinity; // 1/0 的结果
-Infinity; // -1/0 的结果
NaN; // 0/0 的结果

// 也有布尔值。
true;
false;

// 可以通过单引号或双引号来构造字符串。
'abc';
"Hello, world";

// 用！来取非
!true; // = false
!false; // = true

// 相等 ===
1 === 1; // = true
2 === 1; // = false

// 不等 !=
1 !== 1; // = false
2 !== 1; // = true

// 更多的比较操作符 
1 < 10; // = true
1 > 10; // = false
2 <= 2; // = true
2 >= 2; // = true

// 字符串用+连接
"Hello " + "world!"; // = "Hello world!"

// 字符串也可以用 < 、> 来比较
"a" < "b"; // = true

// 使用“==”比较时会进行类型转换...
"5" == 5; // = true
null == undefined; // = true

// ...除非你是用 ===
"5" === 5; // = false
null === undefined; // = false 

// ...但会导致奇怪的行为
13 + !0; // 14
"13" + !0; // '13true'

// 你可以用`charAt`来得到字符串中的字符
"This is a string".charAt(0);  // = 'T'

// ...或使用 `substring` 来获取更大的部分。
"Hello world".substring(0, 5); // = "Hello"

// `length` 是一个属性，所以不要使用 ().
"Hello".length; // = 5

// 还有两个特殊的值：`null`和`undefined`
null;      // 用来表示刻意设置的空值
undefined; // 用来表示还没有设置的值(尽管`undefined`自身实际是一个值)

// false, null, undefined, NaN, 0 和 "" 都是假的；其他的都视作逻辑真
// 注意 0 是逻辑假而  "0"是逻辑真，尽管 0 == "0"。

///////////////////////////////////
// 2. 变量、数组和对象

// 变量需要用`var`关键字声明。Javascript是动态类型语言，
// 所以你无需指定类型。 赋值需要用 `=` 
var someVar = 5;

// 如果你在声明时没有加var关键字，你也不会得到错误...
someOtherVar = 10;

// ...但是此时这个变量就会在全局作用域被创建，而非你定义的当前作用域

// 没有被赋值的变量都会被设置为undefined
var someThirdVar; // = undefined

// 对变量进行数学运算有一些简写法：
someVar += 5; // 等价于 someVar = someVar + 5; someVar 现在是 10 
someVar *= 10; // 现在 someVar 是 100

// 自增和自减也有简写
someVar++; // someVar 是 101
someVar--; // 回到 100

// 数组是任意类型组成的有序列表
var myArray = ["Hello", 45, true];

// 数组的元素可以用方括号下标来访问。
// 数组的索引从0开始。
myArray[1]; // = 45

// 数组是可变的，并拥有变量 length。
myArray.push("World");
myArray.length; // = 4

// 在指定下标添加/修改
myArray[3] = "Hello";

// javascript中的对象相当于其他语言中的“字典”或“映射”：是键-值对的无序集合。
var myObj = {key1: "Hello", key2: "World"};

// 键是字符串，但如果键本身是合法的js标识符，则引号并非是必须的。
// 值可以是任意类型。
var myObj = {myKey: "myValue", "my other key": 4};

// 对象属性的访问可以通过下标
myObj["my other key"]; // = 4

// ... 或者也可以用 . ，如果属性是合法的标识符
myObj.myKey; // = "myValue"

// 对象是可变的；值也可以被更改或增加新的键
myObj.myThirdKey = true;

// 如果你想要获取一个还没有被定义的值，那么会返回undefined
myObj.myFourthKey; // = undefined

///////////////////////////////////
// 3. 逻辑与控制结构

// 本节介绍的语法与Java的语法几乎完全相同

// `if`语句和其他语言中一样。
var count = 1;
if (count == 3){
    // count 是 3 时执行
} else if (count == 4){
    // count 是 4 时执行
} else {
    // 其他情况下执行 
}

// while循环
while (true) {
    // 无限循环
}

// Do-while 和 While 循环很像 ，但前者会至少执行一次
var input;
do {
    input = getInput();
} while (!isValid(input))

// `for`循环和C、Java中的一样：
// 初始化; 继续执行的条件; 迭代。
for (var i = 0; i < 5; i++){
    // 遍历5次
}

// && 是逻辑与, || 是逻辑或
if (house.size == "big" && house.colour == "blue"){
    house.contains = "bear";
}
if (colour == "red" || colour == "blue"){
    // colour是red或者blue时执行
}

// && 和 || 是“短路”语句，它在设定初始化值时特别有用 
var name = otherName || "default";

// `switch`语句使用`===`检查相等性。
// 在每一个case结束时使用 'break'
// 否则其后的case语句也将被执行。 
grade = 'B';
switch (grade) {
  case 'A':
    console.log("Great job");
    break;
  case 'B':
    console.log("OK job");
    break;
  case 'C':
    console.log("You can do better");
    break;
  default:
    console.log("Oy vey");
    break;
}

///////////////////////////////////
// 4. 函数、作用域、闭包

// JavaScript 函数由`function`关键字定义
function myFunction(thing){
    return thing.toUpperCase();
}
myFunction("foo"); // = "FOO"

// 注意被返回的值必须开始于`return`关键字的那一行，
// 否则由于自动的分号补齐，你将返回`undefined`。
// 在使用Allman风格的时候要注意.
function myFunction()
{
    return // <- 分号自动插在这里
    {
        thisIsAn: 'object literal'
    }
}
myFunction(); // = undefined

// javascript中函数是一等对象，所以函数也能够赋给一个变量，
// 并且被作为参数传递 —— 比如一个事件处理函数：
function myFunction(){
    // 这段代码将在5秒钟后被调用
}
setTimeout(myFunction, 5000);
// 注意：setTimeout不是js语言的一部分，而是由浏览器和Node.js提供的。

// 函数对象甚至不需要声明名称 —— 你可以直接把一个函数定义写到另一个函数的参数中
setTimeout(function(){
    // 这段代码将在5秒钟后被调用
}, 5000);

// JavaScript 有函数作用域；函数有其自己的作用域而其他的代码块则没有。
if (true){
    var i = 5;
}
i; // = 5 - 并非我们在其他语言中所期望得到的undefined

// 这就导致了人们经常使用的“立即执行匿名函数”的模式，
// 这样可以避免一些临时变量扩散到全局作用域去。
(function(){
    var temporary = 5;
    // 我们可以访问修改全局对象（"global object"）来访问全局作用域，
    // 在web浏览器中是`window`这个对象。 
    // 在其他环境如Node.js中这个对象的名字可能会不同。
    window.permanent = 10;
})();
temporary; // 抛出引用异常ReferenceError
permanent; // = 10

// javascript最强大的功能之一就是闭包。
// 如果一个函数在另一个函数中定义，那么这个内部函数就拥有外部函数的所有变量的访问权，
// 即使在外部函数结束之后。
function sayHelloInFiveSeconds(name){
    var prompt = "Hello, " + name + "!";
    // 内部函数默认是放在局部作用域的，
    // 就像是用`var`声明的。
    function inner(){
        alert(prompt);
    }
    setTimeout(inner, 5000);
    // setTimeout是异步的，所以 sayHelloInFiveSeconds 函数会立即退出，
    // 而 setTimeout 会在后面调用inner
    // 然而，由于inner是由sayHelloInFiveSeconds“闭合包含”的，
    // 所以inner在其最终被调用时仍然能够访问`prompt`变量。
}
sayHelloInFiveSeconds("Adam"); // 会在5秒后弹出 "Hello, Adam!"

///////////////////////////////////
// 5. 对象、构造函数与原型

//  对象可以包含方法。
var myObj = {
    myFunc: function(){
        return "Hello world!";
    }
};
myObj.myFunc(); // = "Hello world!"

// 当对象中的函数被调用时，这个函数可以通过`this`关键字访问其依附的这个对象。
myObj = {
    myString: "Hello world!",
    myFunc: function(){
        return this.myString;
    }
};
myObj.myFunc(); // = "Hello world!"

// 但这个函数访问的其实是其运行时环境，而非定义时环境，即取决于函数是如何调用的。
// 所以如果函数被调用时不在这个对象的上下文中，就不会运行成功了。
var myFunc = myObj.myFunc;
myFunc(); // = undefined

// 相应的，一个函数也可以被指定为一个对象的方法，并且可以通过`this`访问
// 这个对象的成员，即使在函数被定义时并没有依附在对象上。
var myOtherFunc = function(){
    return this.myString.toUpperCase();
}
myObj.myOtherFunc = myOtherFunc;
myObj.myOtherFunc(); // = "HELLO WORLD!"

// 当我们通过`call`或者`apply`调用函数的时候，也可以为其指定一个执行上下文。
var anotherFunc = function(s){
    return this.myString + s;
}
anotherFunc.call(myObj, " And Hello Moon!"); // = "Hello World! And Hello Moon!"

// `apply`函数几乎完全一样，只是要求一个array来传递参数列表。
anotherFunc.apply(myObj, [" And Hello Sun!"]); // = "Hello World! And Hello Sun!"

// 当一个函数接受一系列参数，而你想传入一个array时特别有用。
Math.min(42, 6, 27); // = 6
Math.min([42, 6, 27]); // = NaN (uh-oh!)
Math.min.apply(Math, [42, 6, 27]); // = 6

// 但是`call`和`apply`只是临时的。如果我们希望函数附着在对象上，可以使用`bind`。
var boundFunc = anotherFunc.bind(myObj);
boundFunc(" And Hello Saturn!"); // = "Hello World! And Hello Saturn!"

// `bind` 也可以用来部分应用一个函数（柯里化）。
var product = function(a, b){ return a * b; }
var doubler = product.bind(this, 2);
doubler(8); // = 16

// 当你通过`new`关键字调用一个函数时，就会创建一个对象，
// 而且可以通过this关键字访问该函数。
// 设计为这样调用的函数就叫做构造函数。
var MyConstructor = function(){
    this.myNumber = 5;
}
myNewObj = new MyConstructor(); // = {myNumber: 5}
myNewObj.myNumber; // = 5

// 每一个js对象都有一个‘原型’。当你要访问一个实际对象中没有定义的一个属性时，
// 解释器就回去找这个对象的原型。

// 一些JS实现会让你通过`__proto__`属性访问一个对象的原型。
// 这虽然对理解原型很有用，但是它并不是标准的一部分；
// 我们后面会介绍使用原型的标准方式。
var myObj = {
    myString: "Hello world!"
};
var myPrototype = {
    meaningOfLife: 42,
    myFunc: function(){
        return this.myString.toLowerCase()
    }
};

myObj.__proto__ = myPrototype;
myObj.meaningOfLife; // = 42

// 函数也可以工作。
myObj.myFunc() // = "hello world!"

// 当然，如果你要访问的成员在原型当中也没有定义的话，解释器就会去找原型的原型，以此类推。
myPrototype.__proto__ = {
    myBoolean: true
};
myObj.myBoolean; // = true

// 这其中并没有对象的拷贝；每个对象实际上是持有原型对象的引用。
// 这意味着当我们改变对象的原型时，会影响到其他以这个原型为原型的对象。
myPrototype.meaningOfLife = 43;
myObj.meaningOfLife; // = 43

// 我们知道 `__proto__` 并非标准规定，实际上也没有标准办法来修改一个已存在对象的原型。
// 然而，我们有两种方式为指定原型创建一个新的对象。

// 第一种方式是 Object.create，这个方法是在最近才被添加到Js中的，
// 因此并不是所有的JS实现都有这个方法
var myObj = Object.create(myPrototype);
myObj.meaningOfLife; // = 43

// 第二种方式可以在任意版本中使用，不过必须通过构造函数。
// 构造函数有一个属性prototype。但是它 *不是* 构造函数本身的原型；相反，
// 是通过构造函数和new关键字创建的新对象的原型。
MyConstructor.prototype = {
    myNumber: 5,
    getMyNumber: function(){
        return this.myNumber;
    }
};
var myNewObj2 = new MyConstructor();
myNewObj2.getMyNumber(); // = 5
myNewObj2.myNumber = 6
myNewObj2.getMyNumber(); // = 6

// 字符串和数字等内置类型也有通过构造函数来创建的包装类型
var myNumber = 12;
var myNumberObj = new Number(12);
myNumber == myNumberObj; // = true

// 但是它们并非严格等价
typeof myNumber; // = 'number'
typeof myNumberObj; // = 'object'
myNumber === myNumberObj; // = false
if (0){
    // 这段代码不会执行，因为0代表假
}

// 不过，包装类型和内置类型共享一个原型，
// 所以你实际可以给内置类型也增加一些功能，例如对string：
String.prototype.firstCharacter = function(){
    return this.charAt(0);
}
"abc".firstCharacter(); // = "a"

// 这个技巧经常用在“代码填充”中，来为老版本的javascript子集增加新版本js的特性，
// 这样就可以在老的浏览器中使用新功能了。

// 比如，我们知道Object.create并没有在所有的版本中都实现，
// 但是我们仍然可以通过“代码填充”来实现兼容：
if (Object.create === undefined){ // 如果存在则不覆盖
    Object.create = function(proto){
        // 用正确的原型来创建一个临时构造函数
        var Constructor = function(){};
        Constructor.prototype = proto;
        // 之后用它来创建一个新的对象
        return new Constructor();
    }
}
```