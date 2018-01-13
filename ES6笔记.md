# ES6笔记

### 代理 Proxy
ES6规范定义了一个全新的全局构造函数：代理（Proxy）。它可以接受两个参数：目标对象（target）与句柄对象（handler）。

##### 代理对象（target）
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

##### 句柄对象（handler）
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

#### 举个栗子：   “不可能实现的”自动填充对象

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
