---
title: JavaScript面经
date: 2020-04-20 16:18:59
tags:
---

### js判断类型的方法

```bash
1. typeof
// 只能判断原始类型（基本数据类型）
// 判断 null 除外

typeof null     // object
typeof [] // 'object'
typeof {} // 'object'
typeof console.log // 'function'

2. Object.prototype.toString()
Object.prototype.toString.call({})    // '[object Object]'

3. instanceof
// 适用于判断对象类型（引用数据类型）

const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var arr = []
arr instanceof Array    // true

// 对于原始类型来说，你想直接通过 instanceof 来判断类型是不行的，当然我们还是有办法让 instanceof 判断原始类型的

class PrimitiveString {
  static [Symbol.hasInstance](x) {
    return typeof x === 'string'
  }
}
console.log('hello world' instanceof PrimitiveString) // true
```

### == 和 === 有什么区别？

> ==  如果对比双方的类型不一样的话，就会进行类型转换

> === 判断两者类型和值是否都相同

== 判断流程
1. 首先会判断两者类型是否相同。相同的话就是比大小了
2. 类型不相同的话，那么就会进行类型转换
3. 会先判断是否在对比 `null` 和 `undefined`，是的话就会返回 `true`
4. 判断两者类型是否为 `string` 和 `number`，是的话就会将字符串转换为 `number`
5. 判断其中一方是否为 `boolean`，是的话就会把 `boolean` 转为 `number` 再进行判断
6. 判断其中一方是否为 `object` 且另一方为 `string`、`number` 或者 `symbol`，就调用 `object` 的valueOf()方法，用得到的基本类型值按照前面的规则进行比较，如果 `object` 没有valueOf()方法，则调用 toString()
7. 如果有一个操作数是`NaN`，则相等操作符返回 false

```bash
'1' == { name: 'yck' }
        ↓
'1' == '[object Object]'
```

> 面试题：[] == ![]

```bash
1. [] == ![]    // true

根据运算符优先级 ，！ 的优先级是大于 == 的，所以先会执行 ![]
！可将变量转换成boolean类型，null、undefined、NaN以及空字符串('')取反都为true，其余都为false。
!null == true
!undefined == true
!NaN == true
!'' == true

![] == false
!{} == false

总结：
[].toString() ->  '' (返回的是空字符串)，也就是 [] == 0 相当于 '' == 0

[] == ! []   ->   [] == false  ->  [] == 0  ->   '' == 0   ->  0 == 0   ->  true

同理：{} == !{}     // true
{}.toString() ->  NaN(返回的是NaN)

{} == ! {}   ->   {} == false  ->  {} == 0  ->   NaN == 0    ->  false

2. 0 == +[]
// + 操作符是转为Number类型

0 == +[]    ->    0 == 0    ->    true

0 == +{}    ->    0 == NaN  ->    false
```

### call和apply

一、call 和 apply 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 this 的指向；

> apply和call都能继承另外一个对象的方法和属性；

```bash

var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call(foo); // 1

1. call 改变了 this 的指向，指向到 foo
2. bar 函数执行了

```

二、对于 apply、call 二者而言，作用完全一样，只是接受参数的方式不太一样。例如，有一个函数定义如下：

```bash
var func = function(arg1, arg2) {
     
};

就可以通过如下方式来调用：

func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2]);

```

> 其中 this 是你想指定的上下文，他可以是任何一个 JavaScript 对象(JavaScript 中一切皆对象)，call 需要把参数按顺序传递进去，而 apply 则是把参数放在数组里；

#### 模拟实现call和apply

可以从以下几点来考虑如何实现

- 不传入第一个参数，那么默认为 `window`
- 改变了 `this` 指向，让新的对象可以执行该函数。那么思路是否可以变成给新的对象添加一个函数，然后在执行完以后删除？

```bash
// call
Function.prototype.myCall = function (context) {
  var context = context || window
  // 给 context 添加一个属性
  // getValue.call(a, 'yilidan', '24') => a.fn = getValue
  context.fn = this
  // 将 context 后面的参数取出来
  var args = [...arguments].slice(1)
  // getValue.call(a, 'yilidan', '24') => a.fn('yilidan', '24')
  var result = context.fn(...args)
  // 删除 fn
  delete context.fn
  return result
}
```

以上就是 `call` 的思路，`apply` 的实现也类似

``` bash
// apply
Function.prototype.myApply = function (context) {
  var context = context || window
  context.fn = this

  var result
  // 需要判断是否存储第二个参数
  // 如果存在，就将第二个参数展开
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }

  delete context.fn
  return result
}
```

`bind` 和其他两个方法作用也是一致的，只是该方法会返回一个函数。并且我们可以通过 `bind` 实现柯里化。

同样的，也来模拟实现下 `bind`

```bash
// bind
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  var args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```



### 对闭包的理解

> 红宝书(p178)上对于闭包的定义：闭包是指有权访问另外一个函数作用域中的变量的函数。

> MDN 对闭包的定义为：**闭包是指那些能够访问自由变量的函数。** （其中自由变量，指在函数中使用的，但既不是函数参数arguments也不是函数的局部变量的变量，其实就是另外一个函数作用域中的变量。）

> 口述理解：函数 A 返回了一个函数 B，并且函数 B 中使用了函数 A 的变量，函数 B 就被称为闭包。

**闭包的表现形式**

```bash

1. 返回一个函数
function f1() {
  var a = 2
  function f2() {
    console.log(a);     // 2
  }
  return f2;
}
var x = f1();
x();

2. 作为函数参数传递
var a = 1;
function foo(){
  var a = 2;
  function baz(){
    console.log(a);
  }
  bar(baz);
}
function bar(fn){
  // 这就是闭包
  fn();
}
// 输出2，而不是1
foo();

3. 在定时器、事件监听、Ajax请求、跨窗口通信、Web Workers或者任何异步中，只要使用了回调函数，实际上就是在使用闭包。

以下的闭包保存的仅仅是window和当前作用域。

// 定时器
setTimeout(function timeHandler(){
  console.log('111');
}，100)

// 事件监听
$('#app').click(function(){
  console.log('DOM Listener');
})

4. IIFE(立即执行函数表达式)创建闭包, 保存了全局作用域window和当前函数的作用域，因此可以全局的变量。

var a = 2;
(function IIFE(){
  // 输出2
  console.log(a);
})();

```

**如何解决下面的循环输出问题？**

```bash

for(var i = 1; i <= 5; i ++){
  setTimeout(function timer(){
    console.log(i)
  }, 0)
}
输出：6,6,6,6,6
```

为什么会全部输出6？如何改进，让它输出1，2，3，4，5？(方法越多越好)

因为setTimeout为宏任务，由于JS中单线程eventLoop机制，在主线程同步任务执行完后才去执行宏任务，因此循环结束后setTimeout中的回调才依次执行，但输出i的时候当前作用域没有，往上一级再找，发现了i,此时循环已经结束，i变成了6。因此会全部输出6。

解决方法：

```bash

1. 利用IIFE(立即执行函数表达式)当每次for循环时，把此时的i变量传递到定时器中；

for(var i = 1;i <= 5;i++){
  (function(j){
    setTimeout(function timer(){
      console.log(j)
    }, 0)
  })(i)
}

// 1,2,3,4,5

2. 给定时器传入第三个参数, 作为timer函数的第一个函数参数；

for(var i=1;i<=5;i++){
  setTimeout(function timer(j){
    console.log(j)
  }, 0, i)
}

// 1,2,3,4,5

3. 使用ES6中的let

for(let i = 1; i <= 5; i++){
  setTimeout(function timer(){
    console.log(i)
  },0)
}

// 1,2,3,4,5

let使JS发生革命性的变化，让JS有函数作用域变为了块级作用域，用let后作用域链不复存在。代码的作用域以块级为单位{}。

```

### 对原型链的理解

**原型对象和构造函数有何关系**

在JavaScript中，每当定义一个函数数据类型(普通函数、类)时候，都会天生自带一个prototype属性，这个属性指向函数的原型对象。

当函数经过new调用时，这个函数就成为了构造函数，返回一个全新的实例对象，这个实例对象有一个__proto__属性，指向构造函数的原型对象。

**原型链的描述**

 JavaScript通过__proto__指向父类对象，直到指向Object对象为止，这样就行程一个原型指向的链条，即原型链；

- 对象的 hasOwnProperty() 来检查对象自身中是否含有该属性
- 使用 in 检查对象中是否含有某个属性时，如果对象中没有但是原型链中有，也会返回 true

### JS如何实现继承

**组合继承**

```bash

function Parent(value) {
	this.val = value
}
Parent.prototype.getValue = function() {
	console.log(this.val)
}
function Child(value) {
	Parent.call(this, value)
}
Child.prototype = new Parent()

const child = new Child(1)
child.getValue() // 1
child instanceof Parent // true

```

**寄生组合继承（推荐）**

```bash
// 写法一：
function Parent(value) {
	this.val = value
}
Parent.prototype.getValue = function() {
	console.log(this.val)
}
function Child(value) {
	Parent.call(this, value)
}

Child.prototype = Object.create(Parent.prototype, {
	constructor: {
		value: Child,
		enumerable: false,
		writable: true,
		configurable: true
	}
})

const child = new Child(1)
child.getValue() // 1
child instanceof Parent // true

```

```bash
// 写法二：
function Parent5 () {
    this.name = 'parent5';
    this.play = [1, 2, 3];
}

function Child5() {
    Parent5.call(this);
    this.type = 'child5';
}

Child5.prototype = Object.create(Parent5.prototype);
Child5.prototype.constructor = Child5;
  
```

**ES6 extends关键字继承（推荐）**

```bash

class Person {
    constructor(name) {
      this.name = name
    }
    printName() {
      console.log('父类')
    }
    commonFunction() {
      console.log('公共方法')
    }
}
  // 继承父类
class Student extends Person{
  constructor(name, score) {
    super(name)
    this.score = score
  }
  printScore() {
    console.log('子类')
  }
}

let p = new Person('小红')
let s = new Student('小明', 100)
console.log(p.commonMethods===s.commonMethods) // true
  
```

### for of 与 for in 的区别

> for...in循环出的是key，for...of循环出的是value

for in 
```bash
  for(let index in aArray){
    console.log(`${aArray[index]}`)
  }
```

for of
```bash
  for(var value of aArray){
    console.log(value)
  }
```

### new 实现过程
1. 新生成了一个对象
2. 链接到原型
3. 绑定this
4. 返回新对象

```bash
function create() {
    // 创建一个空的对象
    let obj = new Object()
    // 获得构造函数
    let Con = [].shift.call(arguments)
    // 链接到原型
    obj.__proto__ = Con.prototype
    // 绑定 this，执行构造函数
    let result = Con.apply(obj, arguments)
    // 确保 new 出来的是个对象
    return typeof result === 'object' ? result : obj
}
```

### 指定随机数范围（整数）

```bash
function sum(m,n) {
    return Math.floor(Math.random()*(m - n) + n)
}
sum(1, 100)
```

### 深拷贝与浅拷贝

产生的原因

```bash
let a = {
    age: 1
}
let b = a
a.age = 2
console.log(b.age) // 2
```
从上述例子中我们可以发现，如果给一个变量赋值一个对象，那么两者的值会是同一个引用，其中一方改变，另一方也会相应改变。

通常在开发中我们不希望出现这样的问题，我们可以使用浅拷贝来解决这个问题。

#### 浅拷贝使用

```bash
方法一：Object.assign

let a = {
    age: 1
}
let b = Object.assign({}, a)
a.age = 2
console.log(b.age) // 1

方法二：展开运算符 ...

let a = {
    age: 1
}
let b = {...a}
a.age = 2
console.log(b.age) // 1

```

通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就需要使用到深拷贝了

```bash
let a = {
    age: 1,
    jobs: {
        first: 'FE'
    }
}
let b = {...a}
a.jobs.first = 'native'
console.log(b.jobs.first) // native
```

浅拷贝只解决了第一层的问题，如果接下去的值中还有对象的话，那么就又回到刚开始的话题了，两者享有相同的引用。要解决这个问题，我们需要引入深拷贝。

#### 深拷贝

这个问题通常可以通过 JSON.parse(JSON.stringify(object)) 来解决。

```bash
let a = {
    age: 1,
    jobs: {
        first: 'FE'
    }
}
let b = JSON.parse(JSON.stringify(a))
a.jobs.first = 'native'
console.log(b.jobs.first) // FE
```
但是该方法也是有局限性的：

- 会忽略 undefined
- 会忽略 symbol
- 不能序列化函数
- 不能解决循环引用的对象

在通常情况下，复杂数据都是可以序列化的，所以这个函数可以解决大部分问题，并且该函数是内置函数中处理深拷贝性能最快的。当然如果你的数据中含有以上三种情况下，可以使用 [lodash 的深拷贝函数](https://lodash.com/docs##cloneDeep)。

### this的理解

this 是很多人会混淆的概念，但是其实他一点都不难，你只需要记住几个规则就可以了。

```bash
function foo() {
	console.log(this.a)
}
var a = 1
foo()

var obj = {
	a: 2,
	foo: foo
}
obj.foo()

// 以上两者情况 `this` 只依赖于调用函数前的对象，优先级是第二个情况大于第一个情况

// 以下情况是优先级最高的，`this` 只会绑定在 `c` 上，不会被任何方式修改 `this` 指向
var c = new foo()
c.a = 3
console.log(c.a)

// 还有种就是利用 call，apply，bind 改变 this，这个优先级仅次于 new
```

以上几种情况明白了，很多代码中的 this 应该就没什么问题了，下面让我们看看箭头函数中的 this

```bash
function a() {
    return () => {
        return () => {
        	console.log(this)
        }
    }
}
console.log(a()()())
```

箭头函数其实是没有 this 的，这个函数中的 this 只取决于他外面的第一个不是箭头函数的函数的 this。在这个例子中，因为调用 a 符合前面代码中的第一个情况，所以 this 是 window。并且 this 一旦绑定了上下文，就不会被任何代码改变。



### Vue的生命周期

```bash
new Vue()
// 新建Vue实例

beforeCreate    
// 是获取不到 props 或者 data 中的数据的，因为这些数据的初始化都在 initState 中。

created
// 可以访问到之前不能访问到的数据

beforeMount
// 开始创建 VDOM (Virtual DOM)

mounted
// 将 VDOM 渲染为真实 DOM 并且渲染数据

beforeUpdate
// 在数据更新前调用

updated
// 在数据更新后调用

使用keep-alive的情况下
// 用 keep-alive 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 deactivated 钩子函数，命中缓存渲染后会执行 actived 钩子函数。

beforeDestroy
// 销毁组件的钩子函数，适合移除事件、定时器等等

destroyed
// 销毁完毕
```

### Vue中computed和watch的区别

> computed 是计算属性，依赖其他属性计算值，并且 computed 的值有缓存，只有当计算值变化才会返回内容。

> watch 监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作

一般来说需要依赖别的属性来动态获得值的时候可以使用 computed

对于监听到值的变化需要做一些复杂业务逻辑的情况可以使用 watch。



### Vue中响应式的原理

Vue 内部使用了 `Object.defineProperty()` 来实现数据响应式，通过这个函数可以监听到 `set` 和 `get` 的事件。



### 发布-订阅模式

> 发布-订阅模式也叫做观察者模式。通过一对一或者一对多的依赖关系，当对象发生改变时，订阅方都会收到通知

发布订阅模式核心是 `addEventListener` 

在 Vue 中，如何实现响应式也是使用了该模式。对于需要实现响应式的对象来说，在 get 的时候会进行依赖收集，当改变了对象的属性时，就会触发派发更新。



### 什么是BFC

> 块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视化CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

如何创建BFC

- 根元素
- 浮动元素（float 属性不为 none）
- position 为 absolute 或 fixed
- overflow 不为 visible 的块元素
- display 为 inline-block, table-cell, table-caption


