# ES6笔记

### 代理 Proxy
ES6规范定义了一个全新的全局构造函数：代理（Proxy）。它可以接受两个参数：目标对象（target）与句柄对象（handler）。

#### 代理对象（target）
```
var target = {}, handler = {};
var proxy = new Proxy(target, handler);
```
如果调用```proxy.[[Enumerate]]()```，就会返回```target.[[Enumerate]]()```。

执行一条能够触发调用proxy.[[Set]]()方法的语句
```
proxy.color = "pink";
```

```
> target.color
  "pink"
```
它们也不完全相同，你会发现proxy !== target

#### 句柄对象（handler）
句柄对象的方法可以覆写任意代理的内部方法。

举个例子，你可以定义一个handler.set()方法来拦截所有给对象属性赋值的行为：
```
var target = {};
var handler = {
    set: function (target, key, value, receiver) {
        throw new Error("请不要为这个对象设置属性。");
      }
    };
var proxy = new Proxy(target, handler);
> proxy.name = "angelina";
  Error: 请不要为这个对象设置属性。
```       
句柄方法都是可选的，没被句柄拦截的内部方法会直接指向目标

##### 举个栗子：   “不可能实现的”自动填充对象

创建一个Tree()函数来实现以下特性：
```
> var tree = Tree();
> tree
      { }
  > tree.branch1.branch2.twig = "green";
  > tree
      { branch1: { branch2: { twig: "green" } } }
  > tree.branch1.branch3.twig = "yellow";
      { branch1: { branch2: { twig: "green" },
                   branch3: { twig: "yellow" }}}
```
###### Tip：所有中间对象branch1、branch2和branch3都可以自动创建
```
function Tree() {
  return new Proxy({}, handler);
}
var handler = {
  get: function (target, key, receiver) {
    if (!(key in target)) {
      target[key] = Tree();  // 自动创建一个子树
    }
    return Reflect.get(target, key, receiver);
  }
};
```

最后的Reflect.get()调用,只执行委托给目标的默认行为。
  
### const声明变量，但是声明的是常量。
```
const PI = Math.PI
PI = 23 //Module build failed: SyntaxError: /es6/app.js: "PI" is read-only
```
当我们引用第三方库的时声明的变量，用const来声明可以避免未来不小心重命名而导致出现bug:
```
const monent = require('moment')
```
### let块级作用域声明变量
```
let name = 'zach'
while (true) {
    let name = 'obama'
    console.log(name)  //obama
    break
}
console.log(name)  //zach
```
  
### class, extends, super (类，继承，指向父级this)

ES6提供了更接近传统语言的写法，引入了Class（类）这个概念。新的class写法让对象原型的写法更加清晰、更像面向对象编程的语法，也更加通俗易懂。
```
class Animal {
  constructor(){
      this.type = 'animal'
  }
  says(say){
      console.log(this.type + ' says ' + say)
  }
}

let animal = new Animal()
animal.says('hello') //animal says hello

class Cat extends Animal {    //继承Animal所有属性
    constructor(){
        super()  //继承父级this 必须
        this.type = 'cat'
    }
}

let cat = new Cat()
cat.says('hello') //cat says hello
```
### arrow function
这个恐怕是ES6最最常用的一个新特性了，用它来写function比原来的写法要简洁清晰很多:
```
function(i){ return i + 1; } //ES5
(i) => i + 1 //ES6
```
如果方程比较复杂，则需要用{}把代码包起来：
```
function(x, y) { 
    x++;
    y--;
    return x + y;
}
(x, y) => {x++; y--; return x+y}
```
长期以来，JavaScript语言的this对象一直是一个令人头痛的问题，在对象方法中使用this，必须非常小心。
```
class Animal {
    constructor(){
        this.type = 'animal'
    }
    says(say){
        setTimeout(function(){
            console.log(this.type + ' says ' + say)
        }, 1000)
    }
}

 var animal = new Animal()
 animal.says('hi')  //undefined says hi
 ```
 但现在我们有了箭头函数，就不需要这么麻烦了：
 ```
 class Animal {
    constructor(){
        this.type = 'animal'
    }
    says(say){
        setTimeout( () => {
            console.log(this.type + ' says ' + say)
        }, 1000)
    }
}
 var animal = new Animal()
 animal.says('hi')  //animal says hi
 ```
 ###### Tip:当我们使用箭头函数时，函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，它的this是继承外面的，因此内部的this就是外层代码块的this。
 
 ### template string
 这个东西也是非常有用，当我们要插入大段的html内容到文档中时，传统的写法非常麻烦，所以之前我们通常会引用一些模板工具库，比如mustache等等。
 
 大家可以先看下面一段代码：
 ```
 $("#result").append(
  "There are <b>" + basket.count + "</b> " +
  "items in your basket, " +
  "<em>" + basket.onSale +
  "</em> are on sale!"
);
```
我们要用一堆的'+'号来连接文本与变量，而使用ES6的新特性模板字符串后，我们可以直接这么来写：
```
$("#result").append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```
 ### default, rest
 调用animal()方法时忘了传参数，传统的做法就是加上这一句type = type || 'cat' 来指定默认值。
 ```
 function animal(type){
    type = type || 'cat'  
    console.log(type)
}
animal()
```
如果用ES6我们而已直接这么写：
function animal(type = 'cat'){
    console.log(type)
}
animal()

rest语法也很简单，直接看例子：
function animals(...types){
    console.log(types)
}
animals('cat', 'dog', 'fish') //["cat", "dog", "fish"]


### 模块的整体加载
除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。
```
import animal, * as content from './content'  
let says = content.say()
console.log(`The ${content.type} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```

### as实现一键换名
```
import animal, { say, type as animalType } from './content'  
let says = say()
console.log(`The ${animalType} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```
###### Tip：通常星号*结合as一起使用比较合适。

## 终极秘籍
考虑下面的场景：比如content.js一共输出了三个变量（default, say, type）,假如我们的实际项目当中只需要用到type这一个变量，其余两个我们暂时不需要。我们可以只输入一个变量：
```
import { type } from './content' 
```
由于其他两个变量没有被使用，我们希望代码打包的时候也忽略它们，抛弃它们，这样在大项目中可以显著减少文件的体积。

ES6帮我们实现了！

不过，目前无论是webpack还是browserify都还不支持这一功能...

如果你现在就想实现这一功能的话，可以尝试使用rollup.js

他们把这个功能叫做Tree-shaking，哈哈哈，意思就是打包前让整个文档树抖一抖，把那些并未被依赖或使用的东西统统抖落下去。。。

看看他们官方的解释吧：
```
Normally if you require a module, you import the whole thing. ES2015 lets you just import the bits you need, without mucking around with custom builds. It's a revolution in how we use libraries in JavaScript, and it's happening right now.
```

未完待续


希望更全面了解es6伙伴们可以去看阮一峰所著的电子书ECMAScript 6入门
