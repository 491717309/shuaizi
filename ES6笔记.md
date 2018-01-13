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
