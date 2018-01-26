# ES6笔记

### 代理 Proxy
ES6规范定义了一个全新的全局构造函数：代理（Proxy）。它可以接受两个参数：目标对象（target）与句柄对象（handler）。

#### 代理对象（target）
```
var target = {}, handler = {};
var proxy = new Proxy(target, handler);
```
如果调用```proxy.[[Enumerate]]()```，就会返回```target.[[Enumerate]]()```。

执行一条能够触发调用```proxy.[[Set]]()```方法的语句
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
### 箭头函数
箭头函数最直观的三个特点:

+不需要function关键字来创建函数
+省略return关键字
+继承当前上下文的 this 关键字

这个恐怕是ES6最最常用的一个新特性了，用它来写function比原来的写法要简洁清晰很多:
```
function(i){ return i + 1; } //ES5
(i) => i + 1 //ES6
```
如果方程有多个参数的时候，则需要用{}把代码包起来：
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
 
 ### 字符串模版
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
 ### 默认参数default, rest
 调用animal()方法时忘了传参数，传统的做法就是加上这一句type = type || 'cat' 来指定默认值。
 ```
 function animal(type){
    type = type || 'cat'  
    console.log(type)
}
animal()
```
如果用ES6我们而已直接这么写：
```
function action(num = 200) {
    console.log(num)
}
action() //200
action(0) //0
```

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

### includes,repeat
```
includes：判断是否包含然后直接返回布尔值
    let str = 'hahay'
    console.log(str.includes('y')) // true
 repeat: 获取字符串重复n次
    let s = 'he'
    console.log(s.repeat(3)) // 'hehehe'
```

### 数据访问--解构
```
const people = {
    name: 'lux',
    age: 20
}
const { name, age } = people
console.log(`${name} --- ${age}`)
//数组
const color = ['red', 'blue']
const [first, second] = color
console.log(first) //'red'
console.log(second) //'blue'
```
### Spread Operator 展开运算符
ES6中另外一个好玩的特性就是Spread Operator 也是三个点儿...接下来就展示一下它的用途。

组装对象或者数组
```
//数组
const color = ['red', 'yellow']
const colorful = [...color, 'green', 'pink']
console.log(colorful) //[red, yellow, green, pink]

//对象
const alp = { fist: 'a', second: 'b'}
const alphabets = { ...alp, third: 'c' }
console.log(alphabets) //{ "fist": "a", "second": "b", "third": "c"}
```
有时候我们想获取数组或者对象除了前几项或者除了某几项的其他项
```
//数组
const number = [1,2,3,4,5]
const [first, ...rest] = number
console.log(rest) //2,3,4,5
//对象
const user = {
    username: 'lux',
    gender: 'female',
    age: 19,
    address: 'peking'
}
const { username, ...rest } = user
console.log(rest) //{"address": "peking", "age": 19, "gender": "female"}
```
对于 Object 而言，还可以用于组合成新的 Object 。(ES2017 stage-2 proposal) 当然如果有重复的属性名，右边覆盖左边
```
const first = {
    a: 1,
    b: 2,
    c: 6,
}
const second = {
    c: 3,
    d: 4
}
const total = { ...first, ...second }
console.log(total) // { a: 1, b: 2, c: 3, d: 4 }
```
### 生成器（ generator ）
1.能返回一个迭代器的函数
```
// 生成器
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}
// 生成器能像正规函数那样被调用，但会返回一个迭代器
let iterator = createIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

那么问题来了，咱们也不能手动一直调用next()方法，你需要一个能够调用生成器并启动迭代器的方法。就像这样子的
```
function run(taskDef) { //taskDef即一个生成器函数

        // 创建迭代器，让它在别处可用
        let task = taskDef();

        // 启动任务
        let result = task.next();
    
        // 递归使用函数来保持对 next() 的调用
        function step() {
    
            // 如果还有更多要做的
            if (!result.done) {
                result = task.next();
                step();
            }
        }
    
        // 开始处理过程
        step();
    
    }
```
或者使用for...of 循环
```
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```
利用 Generator 函数和for...of循环，实现斐波那契数列的例子
```
function* fibonacci() {
  let [a, b] = [0, 1];
  for (;;) {
    [a, b] = [b, a + b];
    yield b;
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

2.next 方法的参数
yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。
```
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}
var g = f();
g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false } 给reset = true
```

生成器与迭代器最有趣、最令人激动的方面，或许就是可创建外观清晰的异步操作代码。你不必到处使用回调函数，而是可以建立貌似同步的代码，但实际上却使用 yield 来等待异步操作结束。

ES6 没有规定，function关键字与函数名之间的星号，写在哪个位置。这导致下面的写法都能通过。
```
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
```

## Promise 对象
```
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```
上面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log('one')则是立即执行，因此最先输出。
```
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```


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

Normally if you require a module, you import the whole thing. ES2015 lets you just import the bits you need, without mucking around with custom builds. It's a revolution in how we use libraries in JavaScript, and it's happening right now.


未完待续


希望更全面了解es6伙伴们可以去看阮一峰所著的电子书[ECMAScript 6入门](http://es6.ruanyifeng.com/)
