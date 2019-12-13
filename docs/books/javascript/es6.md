---
sidebar: auto
---

# 深入理解ES6
在`ECMAScript6`标准定稿之前，已经开始出现了一些实验性的`转译器(Transpiler)`，例如谷歌的`Traceur`，可以将代码从`ECMAScript6`转换成`ECMAScript5`。但它们大多功能非常有限，或难以插入现有的`JavaScript`构建管道。<br/>
但是，随后出现了的新型转译器`6to5`改变了这一切。它易于安装，可以很好的集成现有的工具中，生成的代码可读，于是就像野火一样逐步蔓延开来，`6to5`也就是现在鼎鼎大名的`Babel`。

## 前言

### ECMAScript6的演变之路
::: tip
JavaScript核心的语言特性是在标准`ECMA-262`中被定义，该标准中定义的语言被称作`ECMAScript`，它是`JavaScript`的子集。
:::

* `停滞不前`：逐渐兴起的`Ajax`开创了动态`Web`引用的新时代，而自1999年第三版`ECMA-262`发布以来，`JavaScript`却没有丝毫的改变。
* `转折点`：2007年，`TC-39`委员会将大量规范草案整合在了`ECMAScript4`中，其中新增的语言特性涉足甚广，包括：模块、类、类继承、私有对象成员等众多其它的特性。
* `分歧`：然而`TC-39`组织内部对`ECMAScript4`草案产生了巨大的分歧，部分成员认为不应该一次性在第四版标准中加入过多的新功能，而来自雅虎、谷歌和微软的技术负责人则共同提交了一份`ECMAScript3.1`草案作为下一代`ECMAScript`的可选方案，其中此方案只是对现有标准进行小幅度的增量修改。行为更专注于优化属性特性、支持原生`JSON`以及为已有对象增加新的方法。
* `从未面世的ECMAScript4`：2008年，`JavaScript`创始人`Brendan Eich`宣布`TC-39`委员一方面会将合理推进`ECMAScript3.1`的标准化工作，另一方面会暂时将`ECMAScript4`标准中提出的大部分针对语法及特性的改动搁置。
* `ECMAScript5`：经过标准化的`ECMAScript3.1`最终作为`ECMA-262`第五版与2009年正式发布，同时被命名为`ECMAScript5`。
* `ECMAScript6`：在`ECMAScript5`发布后，`TC-39`委员会于2013年冻结了`ECMAScript6`的草案，不再添加新的功能。2013年`ECMAScript6`草案发布，在进过12个月各方讨论和反馈后。2015年`ECMAScript6`正式发布，并命名为`ECMAScript 2015`。

## 块级作用域绑定
::: tip 引入的目的
一直以来`JavaScript`中的变量生声明机制一直令我们感到困惑，大多数类C语言在声明变量的同时也会创建变量，而在以前的`JavaScript`中，何时创建变量要看如何声明的变量，`ES6`引入块级作用域可以让我们更好的控制作用域。
:::

### var声明和变量提升机制

问：提升机制是什么？<br/>
答：在函数作用域或全局作用域中通过关键字`var`声明的变量，无论实际上是在哪里声明的，都会被当成在当前作用域顶部声明的变量，这就是我们常说的提升机制。<br/>
以下实例代码说明了这种提升机制：
```js
function getValue (condition) {
  if (condition) {
    var value = 'value'
    return value
  } else {
    // 这里可以访问到value，只不过值为undefined
    console.log(value)
    return null
  }
}
getValue(false) // 输出undefined
```
你可以在以上代码中看到，当我们传递了`false`的值，但依然可以访问到`value`这个变量，这是因为在预编译阶段，`JavaScript`引擎会将上面函数的代码修改成如下形式：
```js
function getValue (condition) {
  var value
  if (condition) {
    value = 'value'
    return value
  } else {
    console.log(value)
    return null
  }
}
```

经过以上示例，我们可以发现：变量`value`的声明被提升至函数作用域的顶部，而初始化操作依旧留在原处执行，正因为`value`变量只是声明了而没有赋值，因此以上代码才会打印出`undefined`。


### 块级声明
::: tip
块级声明用于声明在指定的作用于之外无妨访问的变量，块级作用域存在于：函数内部和块中。
:::

`let`声明：
* `let`声明和`var`声明的用法基本相同。
* `let`声明的变量不会被提升。
* `let`不能在同一个作用域中重复声明已经存在的变量，会报错。
* `let`声明的变量作用域范围仅存在于当前的块中，程序进入快开始时被创建，程序退出块时被销毁。

根据`let`声明的规则，改动以上代码后像下面这样：
```js
function getValue (condition) {
  if (condition) {
    // 变量value只存在于这个块中。
    let value = 'value'
    return value
  } else {
    // 访问不到value变量
    console.log(value)
    return null
  }
}
```

`const`声明：`const`声明和`let`声明大多数情况是相同的，唯一的本质区别在于，`const`是用来声明常量的，其声明后的变量值不能再被修改，即意味着：`const`声明必须进行初始化。
```js
const MAX_ITEMS = 30
// 报错
MAX_ITEMS = 50
```

::: warning 注意
我们说的`const`变量值不可变，需要分两种类型来说：
* 值类型：变量的值不能改变。
* 引用类型：变量的地址不能改变，值可以改变。
:::
```js
const num = 23
const arr = [1, 2, 3, 4]
const obj = {
  name: 'why',
  age: 23
}

// 报错
num = 25

// 不报错
arr[0] = 11
obj.age = 32
console.log(arr) // [11, 2, 3, 4]
console.log(obj) // { name: 'why', age: 32 }

// 报错
arr = [4, 3, 2, 1]
```

### 暂时性死区
因为`let`和`const`声明的变量不会进行声明提升，所以在`let`和`const`变量声明之前任何访问(即使是`typeof`也不行)此变量的操作都会引发错误：
```js
if (condition) {
  // 报错
  console.log(typeof value)
  let value = 'value'
}
```
问：为什么会报错？<br/>
答：`JavaScript`引擎在扫描代码发现变量声明时，要么将它们提升至作用域的顶部(`var`声明)，要么将声明放在`TDZ`(暂时性死区)中(`let`和`const`)。访问`TDZ`中的变量会触发错误，只有执行变量声明语句之后，变量才会从`TDZ`中移出，随后才能正常访问。

### 全局块作用域绑定
我们都知道：如果我们在全局作用域下通过`var`声明一个变量，那么这个变量会挂载到全局对象`window`上：
```js
var name = 'why'
console.log(window.name) // why
```
但如果我们使用`let`或者`const`在全局作用域下创建一个新的变量，这个变量不会添加到`window`上。
```js
const name = 'why'
console.log('name' in window) // false
```

### 块级绑定最佳实践的进化
在`ES6`早期，人们普遍认为应该默认使用`let`来代替`var`，这是因为对于开发者而言，`let`实际上与他们想要的`var`一样，直接替换符合逻辑。但随着时代的发展，另一种做法也越来越普及：默认使用`const`，只有确定变量的值会在后续需要修改时才会使用`let`声明，因为大部分变量在初始化后不应再改变，而预料以外的变量值改变是很多`bug`的源头。

## 字符串

### 模块字面量
::: tip
模板字面量是扩展`ECMAScript`基础语法的语法糖，其提供了一套生成、查询并操作来自其他语言里内容的`DSL`，且可以免受`XSS`注入攻击和`SQL`注入等等。
:::

在`ES6`之前，`JavaScript`一直以来缺少许多特性：
* **多行字符串**：一个正式的多行字符串的概念。
* **基本的字符串格式化**：将变量的值嵌入字符串的能力。
* **HTML转义**：向`HTML`插入经过安全转换后的字符串的能力。

而在`ECMAScript 6`中，通过模板字面量的方式多以上问题进行了填补，一个最简单的模板字面量的用法如下：
```js
const message = `hello,world!`
console.log(message) // hello,world!
console.log(typeof message) // string
```
一个需要注意的地方就是，如果我们需要在字符串中使用反撇号，需要使用`\`来进行转义，如下：
```js
const message = `\`hello\`,world!`
console.log(message) // `hello`,world!
```

#### 多行字符串
自`JavaScript`诞生起，开发者们就一直在尝试和创建多行字符串，以下是`ES6`之前的方法：
::: tip
在字符串的新行最前方加上`\`可以承接上一行代码，可以利用这个小`bug`来创建多行字符串。
:::
```js
const message = 'hello\
,world!'
console.log(message) // hello,world
```

在`ES6`之后，我们可以使用模板字面量，在里面直接还行就可以创建多行字符串，如下：
::: tip
在模板字面量中，即反撇号中所有空白字符都属于字符串的一部分。
:::
```js
const message = `hello
,world!`
console.log(message) // hello
                     // ,world!
```

#### 字符串占位符
::: tip
模板字面量于普通字符串最大的区别是模板字符串中的占位符功能，其中占位符中的内容，可以是任意合法的`JavaScript`表达式，例如：变量，运算式，函数调用，设置是另外一个模板字面量。
:::
```js
const age = 23
const name = 'why'
const message = `Hello ${name}, you are ${age} years old!`
console.log(message) // Hello why, you are 23 years old!
```
模板字面量嵌套：
```js
const name = 'why'
const message = `Hello, ${`my name is ${name}`}.`
console.log(message) // Hello, my name is why.
```

#### 标签模板
::: tip
标签指的是在模板字面量第一个反撇号前方的标注的字符串，每一个模板标签都可以执行模板字面量上的转换并返回最终的字符串值。
:::
```js
// tag就是`Hello world!`模板字面量的标签模板
const message = tag`Hello world!`
```
标签可以是一个函数，标签函数通常使用不定参数特性来定义占位符，从而简化数据处理的过程，就像下面这样：
```js
function tag(literals, ...substitutions) {
  // 返回一个字符串
}
const name = 'why'
const age = 23
const message = tag`${name} is ${age} years old!`
```
其中`literals`是一个数组，它包含：
* 第一个占位符前的空白字符串：""。
* 第一个、第二个占位符之间的字符串：" is "。
* 第二个占位符后的字符串：" years old!"

`substitutions`也是一个数组：
* 数组第一项为：`name`的值，即：`why`。
* 数组第二项为：`age`的值，即：`23`。

通过以上规律我们可以发现：
* `literals[0]`始终代表字符串的开头。
* `literals`总比`substitutions`多一个。

我们可以通过以上这种模式，将`literals`和`substitutions`这两个数组交织在一起重新组成一个字符串，来模拟模板字面量的默认行为，像下面这样：
```js
function tag(literals, ...substitutions) {
  let result = ''
  for (let i = 0; i< substitutions.length; i++) {
    result += literals[i]
    result += substitutions[i]
  }

  // 合并最后一个
  result += literals[literals.length - 1]
  return result
}
const name = 'why'
const age = 23
const message = tag`${name} is ${age} years old!`
console.log(message) // why is 23 years old!
```

#### 原生字符串信息
::: tip
通过模板标签可以访问到字符串转义被转换成等价字符串前的原生字符串。
:::
```js
const message1 = `Hello\nworld`
const message2 = String.raw`Hello\nworld`
console.log(message1) // Hello
                      // world
console.log(message2) // Hello\nworld
```

## 函数

### 形参默认值

在`ES6`之前，你可以会通过以下这种模式创建函数并为参数提供默认值：
```js
function makeRequest (url, timeout, callback) {
  timeout = timeout || 2000
  callback = callback || function () {}
}
```
代码分析：在以上示例中，`timeout`和`callback`是可选参数，如果不传入则会使用逻辑或操作符赋予默认值。然而这种方式也有一定的缺陷，如果我们想给`timeout`传递值为`0`，虽然这个值是合法的，但因为有或逻辑运算符的存在，最终还是为`timeout`赋值`2000`。<br/>

针对以上情况，我们应该通过一种更安全的做法(使用`typeof`)来重写一下以上示例：
```js
function makeRequest (url, timeout, callback) {
  timeout = typeof timeout !== 'undefined' ? timeout : 2000
  callback = typeof callback !== 'undefined' ? callback : function () {} 
}
```
代码分析：尽管以上方法更安全一些，但我们任然需要额外的撰写更多的代码来实现这种非常基础的操作。针对以上问题，`ES6`简化了为形参提供默认值的过程，就像下面这样：
::: tip
对于默认参数而言，除非不传或者主动传递`undefined`才会使用参数默认值(如果传递`null`，这是一个合法的参数，不会使用默认值)。
:::
```js
function makeRequest (url, timeout = 2000, callback = function () {}) {
  // todo
}
// 同时使用timeout和callback默认值
makeRequest('https://www.taobao.com')
// 使用callback默认值
makeRequest('https://www.taobao.com', 500)
// 不使用默认值
makeRequest('https://www.taobao.com', 500, function (res) => {
  console.log(res)
})
```
#### 形参默认值对arguments对象的影响
在`ES5`非严格模式下，如果修改参数的值，这些参数的值会同步反应到`arguments`对象中，如下：
```js
function mixArgs(first, second) {
  console.log(arguments[0]) // A
  console.log(arguments[1]) // B
  first = 'a'
  second = 'b'
  console.log(arguments[0]) // a
  console.log(arguments[1]) // b
}
mixArgs('A', 'B')
```
而在`ES5`严格模式下，修改参数的值不再反应到`arguments`对象中，如下：
```js
function mixArgs(first, second) {
  'use strict'
  console.log(arguments[0]) // A
  console.log(arguments[1]) // B
  first = 'a'
  second = 'b'
  console.log(arguments[0]) // A
  console.log(arguments[1]) // B
}
mixArgs('A', 'B')
```

对于使用了`ES6`的形参默认值，`arguments`对象的行为始终保持和`ES5`严格模式一样，无论当前是否为严格模式，即：`arguments`总是等于最初传递的值，不会随着参数的改变而改变，总是可以使用`arguments`对象将参数还原为最初的值，如下：
```js
function mixArgs(first, second = 'B') {
  console.log(arguments.length) // 1
  console.log(arguments[0])      // A
  console.log(arguments[1])      // undefined
  first = 'a'
  second = 'b'
  console.log(arguments[0])      // A
  console.log(arguments[1])      // undefined
}
// arguments对象始终等于传递的值，形参默认值不会反映在arguments上
mixArgs('A')
```

#### 默认参数表达式
::: tip
函数形参默认值，除了可以是原始值的默认值，也可以是表达式，即：变量，函数调用也是合法的。
:::
```js
function getValue () {
  return 5
}
function add (first, second = getValue()) {
  return first + second
}
console.log(add(1, 1)) // 2
console.log(add(1))    // 6
```
代码分析：当我们第一次调用`add(1,1)`函数时，由于未使用参数默认值，所以`getValue`并不会调用。只有当我们使用了`second`参数默认值的时候`add(1)`，`getValue`函数才会被调用。<br/>

正因为默认参数是在函数调用时求值，所以我们可以在后定义的参数表达式中使用先定义的参数，即可以把先定义的参数当做变量或者函数调用的参数，如下：
```js
function getValue(value) {
  return value + 5
}
function add (first, second = first + 1) {
  return first + second
}
function reduce (first, second = getValue(first)) {
  return first - second
}
console.log(add(1))     // 3
console.log(reduce(1))  // -5
```
#### 默认参数的暂时性死区
在前面已经提到过`let`和`const`存在暂时性死区，即：在`let`和`const`变量声明之前尝试访问该变量会触发错误。相同的道理，在函数默认参数中也存在暂时性死区，如下：
```js
function add (first = second, second) {
  return first + second
}
add(1, 1)         // 2
add(undefined, 1) // 抛出错误
```
代码分析：在第一次调用`add(1,1)`时，我们传递了两个参数，则`add`函数不会使用参数默认值；在第二次调用`add(undefined, 1)`时，我们给`first`参数传递了`undefined`，则`first`参数使用参数默认值，而此时`second`变量还没有初始化，所以被抛出错误。

### 不定参数
::: tip
在`JavaScript`的函数语法规定：无论函数已定义的命名参数有多少个，都不限制调用时传入的实际参数的数量。在`ES6`中，当传入更少的参数时，使用参数默认值来处理；当传入更多数量的参数时，使用不定参数来处理。
:::

我们以`underscore.js`库中的`pick`方法为例：
::: tip
`pick`方法的用法是：给定一个对象，返回指定属性的对象的副本。
:::
```js
function pick(object) {
  let result = Object.create(null)
  for (let i = 1, len = arguments.length; i < len; i++) {
    let item = arguments[i]
    result[item] = object[item]
  }
  return result
}
const book = {
  title: '深入理解ES6',
  author: '尼古拉斯',
  year: 2016
}
console.log(pick(book, 'title', 'author')) // { title: '深入理解ES6', author: '尼古拉斯' }
```
代码分析：
* 不容易发现这个函数可以接受任意数量的参数。
* 当需要查找待拷贝的属性的时候，不得不从索引1开始。

在`ES6`中提供了不定参数，我们可以使用不定参数的特性来重写`pick`函数：
```js
function pick(object, ...keys) {
  let result = Object.create(null)
  for (let i = 0, len = keys.length; i < len; i++) {
    let item = keys[i]
    result[item] = object[item]
  }
  return result
}
const book = {
  title: '深入理解ES6',
  author: '尼古拉斯',
  year: 2016
}
console.log(pick(book, 'title', 'author')) // { title: '深入理解ES6', author: '尼古拉斯' }
```

#### 不定参数的限制
不定参数在使用的过程中有几点限制：
* 一个函数最多只能有一个不定参数。
* 不定参数一定要放在所有参数的最后一个。
* 不能在对象字面量`setter`之中使用不定参数。

```js
// 报错，只能有一个不定参数
function add(first, ...rest1, ...rest2) {
  console.log(arguments)
}
// 报错，不定参数只能放在最后一个参数
function add(first, ...rest, three) {
  console.log(arguments)
}
// 报错，不定参数不能用在对象字面量`setter`之中
const object = {
  set name (...val) {
    console.log(val)
  }
}
```

### 展开运算符
在`ES6`的新功能中，展开运算符和不定参数是最为相似的，不定参数可以让我们指定多个各自独立的参数，并通过整合后的数组来访问；而展开运算符可以让你指定一个数组，将它们打散后作为各自独立的参数传入函数。<br/>
在`ES6`之前，我们如果使用`Math.max`函数比较一个数组中的最大值，则需要像下面这样使用：
```js
const arr = [4, 10, 5, 6, 32]
console.log(Math.max.apply(Math, arr)) // 32
```
代码分析：在`ES6`之前使用这种方式是没有任何问题的，但关键的地方在于我们要借用`apply`方法，而且要特别小心的处理`this`(第一个参数)，在`ES6`中我们有更加简单的方式来达到以上的目的：
```js
const arr = [4, 10, 5, 6, 32]
console.log(Math.max(...arr)) // 32
```

### 函数name属性
问：为什么`ES6`会引入函数的`name`属性。
答：在`JavaScript`中有多重定义函数的方式，因而辨别函数就是一项具有挑战性的任务，此外匿名函数表达式的广泛使用也加大了调试的难度，为了解决这些问题，在`ESCAScript 6`中为所有函数新增了`name`属性。

#### 常规name属性
在函数声明和匿名函数表达式中，函数的`name`属性相对来说是固定的：
```js
function doSomething () {
  console.log('do something')
}
let doAnotherThing = function () {
  console.log('do another thing')
}
console.log(doSomething.name)    // doSomething
console.log(doAnotherThing.name) // doAnotherThing
```

#### name属性的特殊情况
尽管确定函数声明和函数表达式的名称很容易，但还是有一些其他情况不是特别容易识别：
* 匿名函数表达式显示提供函数名的情况：函数名称本身比函数本身被赋值的变量的权重高。
* 对象字面量：在不提供函数名称的情况下，取对象字面量的名称；提供函数名称的情况下就是此名称
* 属性的`getter`和`setter`：在对象上存在`get + 属性`的`get`或者`set`方法。
* 通过`bind`：通过`bind`函数创建的函数，`name`为会带有`bound`前缀
* 通过构造函数：函数名称固定为`anonymous`。

```js
let doSomething = function doSomethingElse () {
  console.log('do something else')
}
let person = {
  // person对象上存在name为get firstName的方法
  get firstName () {
    return 'why'
  },
  sayName: function () {
    console.log('why')
  },
  sayAge: function sayNewAge () {
    console.log(23)
  }
}
console.log(doSomething.name)         // doSomethingElse
console.log(person.sayName.name)      // sayName
console.log(person.sayAge.name)       // sayNewAge
console.log(doSomething.bind().name)  // bound doSomethingElse
console.log(new Function().name)      // anonymous
```

### 函数的多种用途
在`JavaScript`中函数具有多重功能，可以结合`new`使用，函数内的`this`值指向一个新对象，函数最终会返回这个新对象，如下：
```js
function Person (name) {
  this.name = name
}
const person = new Person('why')
console.log(person.toString()) // [object Object]
```

在`ES6`中，函数有两个不同的内部方法，分别是：
::: tip
具有`[[Construct]]`方法的函数被称为构造函数，但并不是所有的函数都有`[[Construct]]`方法，例如：箭头函数。
:::
* `[[Call]]`：当通过`new`关键字调用函数时，执行的是`[[Construct]]`函数，它负责创建一个新对象，然后再执行函数体，将`this`绑定到实例上。
* `[[Construct]]`：如果不通过`new`关键字进行调用函数，则执行`[[Call]]`函数，从而直接执行代码中的函数体。

在`ES6`之前，如果要判断一个函数是否通过`new`关键词调用，最流行的方法是使用`instanceof`来判断，例如：
```js
function Person (name) {
  if (this instanceof Person) {
    this.name = name
  } else {
    throw new Error('必须通过new关键词来调用Person')
  }
}
const person = new Person('why')
const notPerson = Person('why') // 抛出错误
```
代码分析：这段代码中，首先会判断`this`的值，看是否是`Person`的实例，如果是则继续执行，如果不是则抛出错误。通常来说这种做法是正确的，但是也不是十分靠谱，有一种方式可以不依赖`new`关键词也可以把`this`绑定到`Person`的实例上，如下：
```js
function Person (name) {
  if (this instanceof Person) {
    this.name = name
  } else {
    throw new Error('必须通过new关键词来调用Person')
  }
}
const person = new Person('why')
const notPerson = Person.call(person, 'why') // 不报错，有效
```

为了解决判断函数是否通过`new`关键词调用的问题，`ES6`引入了`new.target`这个元属性<br/>
问：什么是元属性？<br/>
答：元属性是指非对象的属性，其可以提供非对象目标的补充信息。当调用函数的`[[Construct]]`方法时，`new.target`被赋值为`new`操作符的目标，通常是新创建对象的实例，也就是函数体内`this`的构造函数；如果调用`[[Call]]`方法，则`new.target`的值为`undefined`。

根据以上`new.target`的特点，我们改写一下上面的代码：
::: warning
在函数外使用`new.target`是一个语法错误。
:::
```js
function Person (name) {
  if (typeof new.target !== 'undefined') {
    this.name = name
  } else {
    throw new Error('必须通过new关键词来调用Person')
  }
}
const person = new Person('why')
const notPerson = Person.call(person, 'why') // 抛出错误
```

### 块级函数
在`ECMAScript 3`和早期版本中，在代码块中声明一个块级函数严格来说是一个语法错误，但是所有的浏览器任然支持这个热性，却又因为浏览器的差异导致支撑程度稍有不同，所以**最好不要使用这个特性，如果要用可以使用匿名函数表达式**。
```js
// ES5严格模式下，在代码块中声明一个函数会报错
// 在ES6下，因为有了块级作用域的概念，所以无论是否处于严格模式，都不会报错。
// 但在ES6中，当处于严格模式时，会将函数声明提升至当前块级作用域的顶部
// 当处于非严格模式时，提升至外层作用域
'use strict'
if (true) {
  function doSomething () {
    console.log('do something')
  }
}
```

### 箭头函数
在`ES6`中，箭头函数是其中最有趣的新增特性之一，箭头函数是一种使用箭头`=>`定义函数的新语法，但它和传统的`JavaScript`函数有些许不同：
* **没有this、super、arguments和new.target绑定**：箭头函数中的`this`、`super`、`arguments`和`new.target`这些值又外围最近一层非箭头函数所决定。
* **不能通过new关键词调用**：因为箭头函数没有`[[Construct]]`函数，所以不能通过`new`关键词进行调用，如果使用`new`进行调用会抛出错误。
* **没有原型**：因为不会通过`new`关键词进行调用，所以没有构建原型的需要，也就没有了`prototype`这个属性。
* **不可以改变this的绑定**：在箭头函数的内部，`this`的之不可改变(即不能通过`call`、`apply`或者`bind`等方法来改变)。
* **不支持argument对象**：箭头函数没有`arguments`绑定，所以必须使用命名参数或者不定参数这两种形式访问参数。
* **不支持重复的命名参数**：无论是否处于严格模式，箭头函数都不支持重复的命名参数。

#### 箭头函数的语法
::: tip
箭头函数的语法多变，根据实际的使用场景有多种形式。所有变种度由函数参数、箭头和函数体组成。
:::
表现形式之一：
```js
// 表现形式之一：没有参数
let reflect = () => 5
// 相当于
let reflect = function () {
  return 5
}
```

表现形式之二：
```js
// 表现形式之二：返回单一值
let reflect = value => value
// 相当于
let reflect = function (value) {
  return value
}
```

表现形式之三：
```js
// 表现形式之三：多个参数
let reflect = (val1, val2) => val1 + val2
// 或者
let reflect = (val, val2) => {
  return val1 + val2
}
// 相当于
let reflect = function (val1, val2) {
  return val1 + val2
}
```

表现形式之四：
```js
// 表现形式之四：返回字面量
let reflect = (id) => ({ id: id, name: 'why' })
// 相当于
let reflect = function (id) {
  return {
    id: id,
    name: 'why'
  }
}
```

#### 箭头函数和数组
::: tip
箭头函数的语法简洁，非常适用于处理数组。
:::
```js
const arr = [1, 5, 3, 2]
// 非箭头函数排序写法
arr.sort(function(a, b) {
  return a -b
})
// 箭头函数排序写法
arr.sort((a, b) => a - b)
```


### 尾调用优化
::: tip
尾调用指的是函数作为另一个函数的最后一条语句被调用。
:::
尾调用示例：
```js
function doSomethingElse () {
  console.log('do something else')
}
function doSomething () {
  return doSomethingElse()
}
```

在`ECMAScript 5`的引擎中，尾调用的实现与其他函数调用的实现类似：创建一个新的栈帧，将其推入调用栈来表示函数调用，即意味着：在循环调用中，每一个未使用完的栈帧度会被保存在内存中，当调用栈变得过大时会造成程序问题。<br/>

针对以上可能会出现的问题，`ES6`缩减了严格模式下尾调用栈的大小，当全部满足以下条件，尾调用不再创建新的栈帧，而是清除并重用当前栈帧：
* **尾调用不访问当前栈帧的变量(函数不是一个闭包。)**
* **尾调用不是最后一条语句**
* **尾调用的结果作为函数返回**
满足以上条件的一个尾调用示例：
```js
'use strict'
function doSomethingElse () {
  console.log('do something else')
}
function doSomething () {
  return doSomethingElse()
}
```

不满足以上条件的尾调用示例：
```js
function doSomethingElse () {
  console.log('do something else')
}
function doSomething () {
  // 无法优化，没有返回
  doSomethingElse()
}
function doSomething () {
  // 无法优化，返回值又添加了其它操作
  return 1 + doSomethingElse()
}
function doSomething () {
  // 可能无法优化
  let result = doSomethingElse
  return result
}
function doSomething () {
  let number = 1
  let func = () => number
  // 无法优化，该函数是一个闭包
  return func()
}
```

::: tip
递归函数是其最主要的应用场景，当递归函数的计算量足够大，尾调用优化可以大幅提升程序的性能。
:::
```js
// 优化前
function factorial (n) {
  if (n <= 1) {
    return 1
  } else {
    // 无法优化
    return n * factorial (n - 1)
  }
}

// 优化后
function factorial (n, p = 1) {
  if (n <= 1) {
    return 1 * p
  } else {
    let result = n * p
    return factorial(n -1, result)
  }
}
```

## 对象的扩展

### 对象字面量的扩展
对象字面量扩展包含两部分：
* **属性初始值的简写**：当对象的属性和本地变量同名时，不必再写冒号和值，简单的只写属性即可。
* **对象方法的简写**： 消除了冒号和`function`关键字。
* **可计算属性名**：在定义对象时，对象的属性值可通过变量来计算。
::: tip
通过对象方法简写语法创建的方法有一个`name`属性，其值为小括号前的名称。
:::
```js
const name = 'why'
const firstName = 'first name'
const person = {
  name,
  [firstName]: 'ABC',
  sayName () {
    console.log(this.name)
  }
  
}
// 相当于
const name = 'why'
const person = {
  name: name,
  'irst name': 'ABC',
  sayName: function () {
    console.log(this.name)
  }
}
```

### 新增方法

#### Object.is
在使用`JavaScript`比较两个值的时候，我们可能会习惯使用`==`或者`===`来进行判断，使用全等`===`在比较时可以避免触发强制类型转换，所以深受许多人的喜爱。但全等`===`也并非是完全准确的，例如：
`+0===-0`会返回`true`，`NaN===NaN`会返回`false`。针对以上情况，`ES6`引入了`Object.is`方法来弥补。
```js
// ===和Object.is大多数情况下结果是相同的，只有极少数结果不同
console.log(+0 === -0)            // true
console.log(Object.is(+0, -0))    // false
console.log(NaN === NaN)          // false
console.log(Object.is(NaN, NaN))  // true
```

#### Object.assign
::: tip 什么是Mixin？
混合`Mixin`是`JavaScript`中实现对象组合最流行的一种模式。在一个`mixin`中，一个对象接受来自另一个对象的属性和方法(`mixin`方法为浅拷贝)。
:::
```js
// mixin方法
function mixin(receiver, supplier) {
  Object.keys(supplier).forEach(function(key) {
    receiver[key] = supplier[key]
  })
  return receiver
}
const person1 = {
  age: 23,
  name: 'why'
}
const person2 = mixin({}, person1)
console.log(person2) // { age: 23, name: 'why' }
```
由于这种混合模式非常流行，所以`ES6`引入了`Object.assign`方法来实现相同的功能，这个方法接受一个接受对象和任意数量的源对象，最终返回接受对象。
::: tip
如果源对象中有同名的属性，后面的源对象会覆盖前面源对象中的同名属性。
:::
```js
const person1 = {
  age: 23,
  name: 'why'
}
const person2 = {
  age: 32,
  address: '广东广州'
}
const person3 = Object.assign({}, person1, person2)
console.log(person3) // { age: 32, name: 'why', address: '广东广州' }
```
::: warning
`Object.assign`方法不能复制属性的`get`和`set`。
:::
```js
let receiver = {}
let supplier = {
  get name () {
    return 'why'
  }
}
Object.assign(receiver, supplier)
const descriptor = Object.getOwnPropertyDescriptor(receiver, 'name')
console.log(descriptor.value) // why
console.log(descriptor.get)   // undefined
console.log(receiver)         // { name: 'why' }
```

### 重复的对象字面量属性
在`ECMAScript 5`严格模式下，给一个对象添加重复的属性会触发错误：
```js
'use strict'
const person = {
  name: 'AAA',
  name: 'BBB' // ES5环境触发错误
}
```

但在`ECMAScript 6`中，无论当前是否处于严格模式，添加重复的属性都不会报错，而是选取最后一个取值：
```js
'use strict'
const person = {
  name: 'AAA',
  name: 'BBB' // ES6环境不报错
}
console.log(person) // { name: 'BBB' }
```

### 自有属性枚举顺序
::: tip
`ES5`中未定义对象属性的枚举顺序，由浏览器厂商自行决定。而在`ES6`中严格规定了对象自有属性被枚举时的返回顺序。
:::
**规则**：
* 所有数字键按升序排序。
* 所有字符键按照它们被加入对象的顺序排序。
* 所有`Symbol`键按照它们被加入对象的顺序排序。

根据以上规则，以下这些方法将受到影响：
* `Object.getOwnPropertyNames()`。
* `Reflect.keys()`。
* `Object.assign()`。

不确定的情况：
* `for-in`循环依旧由厂商决定枚举顺序。
* `Object.keys()`和`JSON.stringify()`也同`for-in`循环一样由厂商决定枚举顺序。
```js
const obj = {
  a: 1,
  0: 1,
  c: 1,
  2: 1,
  b: 1,
  1: 1
}
obj.d = 1
console.log(Reflect.keys(obj).join('')) // 012acbd
```
### 增强对象原型
::: tip
`ES5`中，对象原型一旦实例化之后保持不变。而在`ES6`中添加了`Object.setPrototypeOf()`方法来改变这种情况。
:::
```js
const person = {
  sayHello () {
    return 'Hello'
  }
}
const dog = {
  sayHello () {
    return 'wang wang wang'
  }
}
let friend = Object.create(person)
console.log(friend.sayHello())                        // Hello
console.log(Object.getPrototypeOf(friend) === person) // true
Object.setPrototypeOf(friend, dog)
console.log(friend.sayHello())                        // wang wang wang
console.log(Object.getPrototypeOf(friend) === dog)    // true
```
#### 简化原型访问的Super引用
在`ES5`中，如果我们想重写对象实例的方法，又需要调用与它同名的原型方法，可以像下面这样：
```js
const person = {
  sayHello () {
    return 'Hello'
  }
}
const dog = {
  sayHello () {
    return 'wang wang wang'
  }
}
const friend = {
  sayHello () {
    return Object.getPrototypeOf(this).sayHello.call(this) + '!!!'
  }
}
Object.setPrototypeOf(friend, person)
console.log(friend.sayHello())                        // Hello!!!
console.log(Object.getPrototypeOf(friend) === person) // true
Object.setPrototypeOf(friend, dog)
console.log(friend.sayHello())                        // wang wang wang!!!
console.log(Object.getPrototypeOf(friend) === dog)    // true
```
代码分析：要准确记住如何使用`Object.getPrototypeOf()`和`xx.call(this)`方法来调用原型上的方法实在是有点复杂。而且存在多继承的情况下，`Object.getPrototypeOf()`会出现问题。<br/>
根据以上问题，`ES6`引入了`super`关键字，其中`super`相当于指向对象原型的指针，所以以上代码可以修改如下：
::: tip
`super`关键字只出现在对象简写方法里，普通方法中使用会报错。
:::
```js
const person = {
  sayHello () {
    return 'Hello'
  }
}
const dog = {
  sayHello () {
    return 'wang wang wang'
  }
}
const friend = {
  sayHello () {
    return super.sayHello.call(this) + '!!!'
  }
}
Object.setPrototypeOf(friend, person)
console.log(friend.sayHello())                        // Hello!!!
console.log(Object.getPrototypeOf(friend) === person) // true
Object.setPrototypeOf(friend, dog)
console.log(friend.sayHello())                        // wang wang wang!!!
console.log(Object.getPrototypeOf(friend) === dog)    // true
```
### 正式的方法定义
::: tip
在`ES6`之前从未正式定义过"方法"的概念，方法仅仅是一个具有功能而非数据的对象属性。而在`ES6`中正式将方法定义为一个函数，它会有一个内部`[[HomeObject]]`属性来容纳这个方法从属的对象。
:::
```js
const person = {
  // 是方法 [[HomeObject]] = person
  sayHello () {
    return 'Hello'
  }
}
// 不是方法
function sayBye () {
  return 'goodbye'
}
```
根据以上`[[HomeObject]]`的规则，我们可以得出`super`是如何工作的：
* 在`[[HomeObject]]`属性上调用`Object.getPrototypeOf()`方法来检索原型的引用。 
* 搜索原型找到同名函数。
* 设置`this`绑定并且调用相应的方法。

```js
const person = {
  sayHello () {
    return 'Hello'
  }
}
const friend = {
  sayHello () {
    return super.sayHello() + '!!!'
  }
}
Object.setPrototypeOf(friend, person)
console.log(friend.sayHello()) // Hello!!!
```
代码分析：
* `friend.sayHello()`方法的`[[HomeObject]]`属性值为`friend`。
* `friend`的原型是`person`。
* `super.sayHello()`相当于`person.sayHello.call(this)`。

## 解构
::: tip
解构是一种打破数据结构，将其拆分为更小部分的过程。
:::

### 为何使用解构功能
在`ECMAScript 5`及其早期版本中，为了从对象或者数组中获取特定数据并赋值给变量，编写了许多看起来同质化的代码：
```js
const person = {
  name: 'AAA',
  age: 23
}
const name = person.name
const age = person.age
```
代码分析：我们必须从`person`对象中提取`name`和`age`的值，并把其值赋值给对应的同名变量，过程极其相似。假设我们要提取许多变量，这种过程会重复更多次，如果其中还包含嵌套结构，只靠遍历是找不到真实信息的。<br/>

针对以上问题，`ES6`引入了解构的概念，按场景可分为：
* 对象解构
* 数组解构
* 混合解构
* 解构参数

### 对象解构
我们使用`ES6`中的对象结构，改写以上示例：
```js
const person = {
  name: 'AAA',
  age: 23
}
const { name, age } = person
console.log(name) // AAA
console.log(age)  // 23
```
::: warning
必须为解构赋值提供初始化程序，同时如果解构右侧为`null`或者`undefined`，解构会发生错误。
:::
```js
// 以下代码为错误示例，会报错
var { name, age }
let { name, age }
const { name, age }
const { name, age } = null
const { name, age } = undefined
```

#### 解构赋值
我们不仅可以在解构时重新定义变量，还可以解构赋值已存在的变量：
```js
const person = {
  name: 'AAA',
  age: 23
}
let name, age
// 必须添加()，因为如果不加，{}代表是一个代码块，而语法规定代码块不能出现在赋值语句的左侧。
({ name, age } = person)
console.log(name) // AAA
console.log(age)  // 23
```

#### 解构默认值
::: tip
使用解构赋值表达式时，如果指定的局部变量名称在对象中不存在，那么这个局部变量会被赋值为`undefined`，此时可以随意指定一个默认值。
:::
```js
const person = {
  name: 'AAA',
  age: 23
}
let { name, age, sex = '男' } = person
console.log(sex) // 男
```

#### 为非同名变量赋值
目前为止我们解构赋值时，带解构的键和带赋值的变量是同名的，但如何为非同名变量解构赋值呢？
```js
const person = {
  name: 'AAA',
  age: 23
}
let { name, age } = person
// 相当于
let { name: name, age: age } = person
```
`let { name: name, age: age } = person`含义是：在`person`对象中取键为`name`和`age`的值，并分别赋值给`name`变量和`age`变量。<br/>
那么，我们根据以上的思路，为非同名变量赋值可以改写成如下形式：
```js
const person = {
  name: 'AAA',
  age: 23
}
let { name: newName, age: newAge } = person
console.log(newName) // AAA
console.log(newAge)  // 23
```

#### 嵌套对象结构
::: tip
解构嵌套对象任然与对象字面量语法相似，只是我们可以将对象拆解成我们想要的样子。
:::
```js
const person = {
  name: 'AAA',
  age: 23,
  job: {
    name: 'FE',
    salary: 1000
  },
  department: {
    group: {
      number: 1000,
      isMain: true
    }
  }
}
let { job, department: { group } } = person
console.log(job)    // { name: 'FE', salary: 1000 }
console.log(group)  // { number: 1000, isMain: true }
```
`let { job, department: { group } } = person`含义是：在`person`中提取键为`job`、在`person`的嵌套对象`department`中提取键为`group`的值，并把其赋值给对应的变量。


### 数组解构
::: tip
数组的解构赋值与对象解构的语法相似，但简单许多，它使用的是数组字面量，且解构操作全部在数组内完成，解构的过程是按值在数组中的位置进行提取的。
:::
```js
const colors = ['red', 'green', 'blue']
let [firstColor, secondColor] = colors
// 按需解构
let [,,threeColor] = colors
console.log(firstColor)   // red
console.log(secondColor)  // green
console.log(threeColor)   // blue
```

与对象一样，解构数组也能解构赋值给已经存在的变量，只是可以不需要像对象一样额外的添加括号：
```js
const colors = ['red', 'green', 'blue']
let firstColor, secondColor
[firstColor, secondColor] = colors
console.log(firstColor)   // red
console.log(secondColor)  // green
```
按以上原理，我们可以轻松扩展一下解构赋值的功能(快速交换两个变量的值)：
```js
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(a); // 2
console.log(b); // 1
```
与对象一样，数组解构也可以设置解构默认值：
```js
const colors = ['red']
const [firstColor, secondColor = 'green'] = colors
console.log(firstColor)   // red
console.log(secondColor)  // green
```

当存在嵌套数组时，我们也可以使用和解构嵌套对象的思路来解决：
```js
const colors = ['red', ['green', 'lightgreen'], 'blue']
const [firstColor, [secondColor]] = colors
console.log(firstColor)   // red
console.log(secondColor)  // green
```

#### 不定参数
::: tip
在解构数组时，不定元素只能放在最后一个，在后面继续添加逗号会导致报错。
:::
在数组解构中，有一个和函数的不定参数相似的功能：在解构数组时，可以使用`...`语法将数组中剩余元素赋值给一个特定的变量：
```js
let colors = ['red', 'green', 'blue']
let [firstColor, ...restColors] = colors
console.log(firstColor) // red
console.log(restColors) // ['green', 'blue']
```
根据以上解构数组中的不定元素的原理，我们可以实现同`concat`一样的数组复制功能：
```js
const colors = ['red', 'green', 'blue']
const concatColors = colors.concat()
const [...restColors] = colors
console.log(concatColors) // ['red', 'green', 'blue']
console.log(restColors)   // ['red', 'green', 'blue']
```

### 解构参数
当我们定一个需要接受大量参数的函数时，通常我们会创建可以可选的对象，将额外的参数定义为这个对象的属性：
```js
function setCookie (name, value, options) {
  options = options || {}
  let path = options.path,
      domain = options.domain,
      expires = options.expires
  // 其它代码
}

// 使用解构参数
function setCookie (name, value, { path, domain, expires } = {}) {
  let { path, domain, expires } = 
  // 其它代码
}
```
代码分析：`{ path, domain, expires } = {}`必须提供一个默认值，如果不提供默认值，则不传递第三个参数或报错：
```js
function setCookie (name, value, { path, domain, expires }) {
  // 其它代码
}
// 报错
setCookie('type', 'js')
// 相当于解构了undefined，所以会报错
{ path, domain, expires } = undefined
```


## Symbol及其Symbol属性
::: tip
在`ES6`之前，`JavaScript`语言只有五种原始类型：`string`、`number`、`boolean`、`null`和`undefiend`。在`ES6`中，添加了第六种原始类型：`Symbol`。
:::
可以使用`typeof`来检测`Symbol`类型：
```js
const symbol = Symbol('Symbol Test')
console.log(typeof symbol) // symbol
```
### 创建Symbol
::: tip
可以通过全局的`Symbol`函数来创建一个`Symbol`。
:::
```js
const firstName = Symbol()
const person = {}
person[firstName] = 'AAA'
console.log(person[firstName]) // AAA
```

可以在`Symbol()`种传递一个可选的参数，可以让我们添加一段文本描述我们创建的`Symbol`，其中文本是存储在内部属性`[[Description]]`中，只有当调用`Symbol`的`toString()`方法时才可以读取这个属性。
```js
const firstName = Symbol('Symbol Description')
const person = {}
person[firstName] = 'AAA'
console.log(person[firstName]) // AAA
console.log(firstName)         // Symbol('Symbol Description')
```

### Symbol的使用方法
::: tip
所有可以使用可计算属性名的地方，都可以使用`Symbol`。
:::
```js
let firstName = Symbol('first name')
let lastName = Symbol('last name')
const person = {
  [firstName]: 'AAA'
}
Object.defineProperty(person, firstName, {
  writable: false
})
Object.defineProperties(person, {
  [lastName]: {
    value: 'BBB',
    writable: false
  }
})
console.log(person[firstName])  // AAA
console.log(person[lastName])   // BBB
```
### Symbol共享体系
::: tip
`ES6`提供了一个可以随时访问的全局`Symbol`注册表来让我们可以创建共享`Symbol`的能力，可以使用`Symbol.for()`方法来创建一个共享的`Symbol`。
:::
```js
// Symbol.for方法的参数，也被用做Symbol的描述内容
const uid = Symbol.for('uid')
const object = {
  [uid]: 12345
}
console.log(person[uid]) // 12345
console.log(uid)         // Symbil(uid)
```
代码分析：
* `Symbol.for()`方法首先会在全局`Symbol`注册变中搜索键为`uid`的`Symbol`是否存在。
* 存在，直接返回已有的`Symbol`。
* 不存在，则创建一个新的`Symbol`，并使用这个键在`Symbol`全局注册变中注册，随后返回新创建的`Symbol`。

还有一个和`Symbol`共享有关的特性，可以使用`Symbol.keyFor()`方法在`Symbol`全局注册变中检索与`Symbol`有关的键，如果存在则返回，不存在则返回`undefined`：
```js
const uid = Symbol.for('uid')
const uid1 = Symbol('uid1')
console.log(Symbol.keyFor(uid))   // uid
console.log(Symbol.keyFor(uid1))  // undefined
```
### Symbol与类型强制转换
::: tip
其它原始类型没有与`Symbol`逻辑相等的值，尤其是不能将`Symbol`强制转换为字符串和数字。
:::
```js
const uid = Symbol.for('uid')
console.log(uid)
console.log(String(uid))
// 报错
uid = uid + ''
uid = uid / 1
```
代码分析：我们使用`console.log()`方法打印`Symbol`，会调用`Symbol`的`String()`方法，因此也可以直接调用`String()`方法输出`Symbol`。然而尝试将`Symbol`和一个字符串拼接，会导致程序抛出异常，`Symbol`也不能和每一个数学运算符混合使用，否则同样会抛出错误。

### Symbol属性检索
`Object.keys()`和`Object.getOwnPropertyNames()`方法可以检索对象中所有的属性名，其中`Object.keys`返回所有可以枚举的属性，`Object.getOwnPropertyNames()`无论属性是否可以枚举都返回，但是这两个方法都无法返回`Symbol`属性。因此`ES6`引入了一个新的方法`Object.getOwnPropertySymbols()`方法。
```js
const uid = Symbol.for('uid')
let object = {
  [uid]: 123
}
const symbols = Object.getOwnPropertySymbols(object)
console.log(symbols.length) // 1
console.log(symbols[0])     // Symbol(uid)
```

### Symbol暴露内部的操作
`ES6`通过在原型链上定义与`Symbol`相关的属性来暴露更多的语言内部逻辑，这些内部操作如下：
* `Symbol.hasInstance`：一个在执行`instanceof`时调用的内部方法，用于检测对象的继承信息。
* `Symbol.isConcatSpreadable`：一个布尔值，用于表示当传递一个集合作为`Array.prototype.concat()`方法的参数时，是否应该将集合内的元素规整到同一层级。
* `Symbol.iterator`：一个返回迭代器的方法。
* `Symbol.match`：一个在调用`String.prototype.match()`方法时调用的方法，用于比较字符串。
* `Symbol.replace`：一个在调用`String.prototype.replace()`方法时调用的方法，用于替换字符串中的子串。
* `Symbol.search`：一个在调用`String,prototype.search()`方法时调用的方法，用于在字符串中定位子串。
* `Symbol.split`：一个在调用`String.prototype.split()`方法时调用的方法，用于分割字符串。
* `Symbol.species`：用于创建派生对象的构造函数。
* `Symbol.toPrimitive`：一个返回对象原始值的方法。
* `Symbol.toStringTag`：一个在调用`Object.prototype.toString()`方法时使用的字符串，用于创建对象描述。
* `Symbol.unscopables`：一个定义了一些不可被`with`语句引用的对象属性名称的对象集合。

重写一个由`well-known Symbol`定义的方法，会导致对象内部的默认行为被改变，从而一个普通对象会变为一个奇异对象。

#### Symbol.hasInstance
::: tip
每一个函数都有`Symbol.hasInstance`方法，用于确定对象是否为函数的实例，并且该方法不可被枚举、不可被写和不可被配置。
:::
```js
function MyObject () {
  // 空函数
}
Object.defineProperty(MyObject, Symbol.hasInstance, {
  value: function () {
    return false
  }
})
let obj = new MyObject()
console.log(obj instanceof MyObject) // false
```
代码分析：使用`Object.defineProperty`方法，在`MyObject`函数上改写`Symbol.hasInstance`，为其定义一个总是返回`false`的新函数，即使`obj`确实是`MyObject`的实例，但依然在进行`instanceof`判断时返回了`false`。
::: warning
注意如果要触发`Symbol.hasInstance`调用，`instanceof`的左操作符必须是一个对象，如果为非对象则会导致`instanceof`始终返回`false`。
:::

#### Symbol.isConcatSpreadable
在`JavaScript`数组中`concat()`方法被用于拼接两个数组：
```js
const colors1 = ['red', 'green']
const colors2 = ['blue']
console.log(colors1.concat(colors2, 'brown')) // ['red', 'green', 'blue', 'brown']
```
在`concat()`方法中，我们传递了第二个参数，它是一个非数组元素。如果`Symbol.isConcatSpreadable`为`true`，那么表示对象有`length`属性和数字键，故它的数值型键会被独立添加到`concat`调用的结果中，它是对象的可选属性，用于增强作用于特定对象类型的`concat`方法的功能，有效简化其默认特性：
```js
const obj = {
  0: 'hello',
  1: 'world',
  length: 2,
  [Symbol.isConcatSpreadable]: true
}
const message = ['Hi'].concat(obj)
console.log(message) // ['Hi', 'hello', 'world']
```

#### Symbol.match，Symbol.replace，Symbol.search，Symbol.split
在`JavaScript`中，字符串与正则表达式经常一起出现，尤其是字符串类型的几个方法，可以接受正则表达式作为参数：
* `match`：确定给定字符串是否匹配正则表示。
* `replace`：将字符串中匹配正则表达式的部分替换为给定的字符串。
* `search`：在字符串中定位匹配正则表示位置的索引。
* `split`：按照匹配正则表达式的元素将字符串进行分割，并将分割结果存入数组中。

在`ES6`之前，以上几个方法无法使用我们自己定义的对象来替代正则表达式进行字符串匹配，而在`ES6`之后，引入了与上述几个方法相对应`Symbol`，将语言内建的`Regex`对象的原生特性完全外包出来。
```js
const hasLengthOf10 = {
  [Symbol.match] (value) {
    return value.length === 10 ? [value] : null
  },
  [Symbol.replace] (value, replacement) {
    return value.length === 10 ? replacement : value
  },
  [Symbol.search] (value) {
    return value.length === 10 ? 0 : -1
  },
  [Symbol.split] (value) {
    return value.length === 10 ? [,] : [value]
  }
}
const message1 = 'Hello world'
const message2 = 'Hello John'
const match1 = message1.match(hasLengthOf10)
const match2 = message2.match(hasLengthOf10)
const replace1 = message1.replace(hasLengthOf10)
const replace2 = message2.replace(hasLengthOf10, 'AAA')
const search1 = message1.search(hasLengthOf10)
const search2 = message2.search(hasLengthOf10)
const split1 = message1.split(hasLengthOf10)
const split2 = message2.split(hasLengthOf10)
console.log(match1)     // null
console.log(match2)     // [Hello John]
console.log(replace1)   // Hello world
console.log(replace2)   // AAA
console.log(search1)    // -1
console.log(search2)    // 0
console.log(split1)     // [Hello John]
console.log(split2)     // [,]
```

#### Symbol.toPrimitive
`Symbol.toPrimitive`方法被定义在每一个标准类型的原型上，并且规定了当对象被转换为原始值时应该执行的操作，每当执行原始值转换时，总会调用`Symbol.toPrimitive`方法并传入一个值作为参数。<br/>
对于大多数标准对象，数字模式有以下特性，根据优先级的顺序排序如下：
* 调用`valueOf()`方法，如果结果为原始值，则返回。
* 否则，调用`toString()`方法，如果结果为原始值，则返回。
* 如果再无可选值，则抛出错误。

同样对于大多数标准对象，字符串模式有以下有限级顺序：
* 调用`toString()`方法，如果结果为原始值，则返回。
* 否则，调用`valueOf()`方法，如果结果为原始值，则返回。
* 如果再无可选值，则抛出错误。

在大多数情况下，标准对象会将默认模式按数字模式处理(除`Date`对象，在这种情况下，会将默认模式按字符串模式处理)，如果自定义了`Symbol.toPrimitive`方法，则可以覆盖这些默认的强制转换行为。
```js
function Temperature (degress) {
  this.degress = degress
}
Temperature.prototype[Symbol.toPrimitive] = function (hint) {
  switch (hint) {
    case 'string':
      return this.degress + '℃'
    case 'number':
      return this.degress
    case 'default':
      return this.deress + ' degress'
  }
}
const freezing = new Temperature(32)
console.log(freezing + '')      // 32 degress
console.log(freezing / 2)       // 16
console.log(String(freezing))   // 32℃
```

#### Symbol.toStringTag
在`JavaScript`中，如果我们同时存在多个全局执行环境，例如在浏览器中一个页面包含`iframe`标签，因为`iframe`和它外层的页面分别代表不同的领域，每一个领域都有自己的全局作用域，有自己的全局对象，在任何领域中创建的数组，都是一个正规的数组。然而，如果将这个数字传递到另外一个领域中，`instanceof Array`语句的检测结果会返回`false`，此时`Array`已经是另一个领域的构造函数，显然被检测的数组不是由这个构造函数创建的。<br/>

针对以上问题，我们很快找到了一个相对来说比较实用的解决方案：
```js
function isArray(value) {
  return Object.prototype.toString.call(value) === '[object Array]'
}
console.log(isArray([])) // true
```
与上述问题有一个类似的案例，在`ES5`之前我们可能会引入第三方库来创建全局的`JSON`对象，而在浏览器开始实现`JSON`全局对象后，就有必要区分`JSON`对象是`JavaScript`环境本身提供的还是由第三方库提供的：
```js
function supportsNativeJSON () {
  return typeof JSON !== 'undefined' && Object.prototype.toString.call(JSON) === '[object JSOn]'
}
```
在`ES6`中，通过`Symbol.toStringTag`这个`Symbol`改变了调用`Object.prototype.toString()`时返回的身份标识，其定义了调用对象的`Object.prototype.toString.call()`方法时返回的值：
```js
function Person (name) {
  this.name = name
}
Person.prototype[Symbol.toStringTag] = 'Person'
const person = new Person('AAA')
console.log(person.toString())                        // [object Person]
console.log(Object.prototype.toString.call(person))   // [object Person]
```

## Set和Map集合
`Set`集合是一种无重复元素的列表，通常用来检测给定的值是否在某个集合中；`Map`集合内含多组键值对，集合中每个元素分别存放着可访问的键名和它对应的值，`Map`集合经常被用来缓存频繁取用的数据。

### ES5中的Set和Map集合
在`ES6`还没有正式引入`Set`集合和`Map`集合之前，开发者们已经开始使用对象属性来模拟这两种集合了：
```js
const set = Object.create(null)
const map = Object.create(null)
set.foo = true
map.bar = 'bar'
// set检查
if (set.foo) {
  console.log('存在')
}
// map取值
console.log(map.bar) // bar
```
以上程序很简单，确实可以使用对象属性来模拟`Set`集合和`Map`集合，但却在实际使用的过程中有诸多的不方便：
* 对象属性名必须为字符串：
```js
const map = Object.create(null)
map[5] = 'foo'
// 本意是使用数字5作为键名，但被自动转换为了字符串
console.log(map['5']) // foo
```
* 对象不能作为属性名：
```js
const map = Object.create(null)
const key1 = {}
const key2 = {}
map[key1] = 'foo'
// 本意是使用key1对象作为属性名，但却被自动转换为[object Object]
// 因此map[key1] = map[key2] = map['[object Object]']
console.log(map[key2]) // foo
```
* 不可控制的强制类型转换：
```js
const map = Object.create(null)
map.count = 1
// 本意是检查count属性是否存在，实际检查的确实map.count属性的值是否非0
if (map.count) {
  console.log(map.count)
}
```
### ES6中的Set集合
::: tip
`Set`集合是一种有序列表，其中含有一些相互独立的非重复值，在`Set`集合中，不会对所存的值进行强制类型转换。
:::
其中`Set`集合涉及到的属性和方法有：
* `Set`构造函数：可以使用此构造函数创建一个`Set`集合。
* `add`方法：可以像`Set`集合中添加一个元素。
* `delete`方法：可以移除`Set`集合中的某一个元素。
* `clear`方法：可以移除`Set`集合中所有的元素。
* `has`方法：判断给定的元素是否在`Set`集合中。
* `size`属性：`Set`集合的长度。

#### 创建Set集合
::: tip
`Set`集合的构造函数可以接受任何可迭代对象作为参数，例如：数组、`Set`集合或者`Map`集合。
:::
```js
const set = new Set()
set.add(5)
set.add('5')
// 重复添加的值会被忽略
set.add(5)
console.log(set.size) // 2
```

#### 移除元素
::: tip
使用`delete()`方法可以移除集合中的某一个值，使用`clear()`方法可以移除集合中所有的元素。
:::
```js
const set = new Set()
set.add(5)
set.add('5')
console.log(set.has(5)) // true
set.delete(5)
console.log(set.has(5)) // false
console.log(set.size)   // 1
set.clear()
console.log(set.size)   // 0
```

#### Set集合的forEach()方法
`Set`集合的`forEach()`方法和数组的`forEach()`方法是一样的，唯一的区别在于`Set`集合在遍历时，第一和第二个参数是一样的。
```js
const set = new Set([1, 2])
set.forEach((value, key, arr) => {
  console.log(`${value} ${key}`)
  console.log(arr === set)
})
// 1 1
// true
// 2 2
// true
```

#### Set集合转换为数组
因为`Set`集合不可以像数组那样通过索引去访问数组元素，最好的做法是将`Set`集合转换为数组。
```js
const set = new Set([1, 2, 3, 4])
// 展开运算符
const arr1 = [...set]
// Array.from方法
const arr2 = Array.from(set)
console.log(arr1) // [1, 2, 3, 4]
console.log(arr2) // [1, 2, 3, 4]
```

#### Weak Set集合
通过以上对`Set`集合的梳理，我们可以发现：只要`Set`实例中的引用存在，垃圾回收机制就不能释放该对象的内存空间，所以我们把`Set`集合看作是一个强引用的集合。<br/>
为了更好的处理`Set`集合的垃圾回收，引入了一个叫`Weak Set`的集合：
::: tip
`Weak Set`集合只支持三种方法：`add`、`has`和`delete`。
:::
```js
const weakSet = new WeakSet()
const key = {}
weakSet.add(key)
console.log(weakSet.has(key)) // true
weakSet.delete(key)
console.log(weakSet.has(key)) // false
```
`Set`集合和`Weak Set`集合有许多共同的特性，但它们之间还是有一定的差别的：
* `Weak Set`集合只能存储对象元素，像其添加非对象元素会导致抛出错误，同理`has()`和`delete()`传递非对象也同样会报错。
* `Weak Set`集合不可迭代，也不暴露任何迭代器。
* `Weak Set`集合不支持`forEach`方法。
* `Weak Set`集合不支持`size`属性。

### ES6中的Map集合
`ES6`中的`Map`类型是一种存储着许多键值对的有序列表，其中的键名和对应的值支持所有的数据类型，键名的等价性判断是通过调用`Object.is`方法来实现的。
```js
const map = new Map()
const key1 = {}
const key2 = {}
map.set(5, 5)
map.set('5', '5')
map.set(key1, key2)
console.log(map.get(5))     // 5
console.log(map.get('5'))   // '5'
console.log(map.get(key1))  // {}
```

#### Map集合支持的方法
与`Set`集合类似，`Map`集合也支持一下集中方法：
* `has`：检出指定的键名是否在`Map`集合中存在。
* `delete`：在`Map`集合中移除指定键名及其对应的值。
* `clear`：移除`Map`集合中所有的键值对。
```js
const map = new Map()
map.set('name', 'AAA')
map.set('age', 23)
console.log(map.size)        // 2
console.log(map.has('name')) // true
console.log(map.get('name')) // AAA
map.delete('name')
console.log(map.has('name')) // false
map.clear()
console.log(map.size)        // 0
```

#### Map集合的初始化方法
在初始化`Map`集合的时候，也可以像`Set`集合传入数组，但此时数组中的每一个元素都是一个子数组，子数组中包含一个键值对的键名和值两个元素。
```js
const map = new Map([['name', 'AAA'], ['age', 23]])
console.log(map.has('name'))  // true
console.log(map.has('age'))   // true
console.log(map.size)         // 2
console.log(map.get('name'))  // AAA
console.log(map.get('age'))   // 23
```

#### Map集合的forEach()方法
`Map`集合中的`forEach()`方法的回调参数和数字类似，每一个参数的解释如下：
* 第一个参数是键名
* 第二个参数是值
* 第三个参数是`Map`集合本身

```js
const map = new Map([['name', 'AAA'], ['age', 23]])
map.forEach((key, value, ownMap) => {
  console.log(`${key} ${value}`)
  console.log(ownMap === map)
})
// name AAA
// true
// age 23
// true
```

#### Weak Map集合
`Weak Map`它是一种存储着许多键值对的无序列表，集合中的键名必须是一个对象，如果使用非对象键名会报错。
::: tip
`Weak Map`集合只支持`set()`、`get()`、`has()`和`delete()`。
:::
```js
const key1 = {}
const key2 = {}
const key3 = {}
const weakMap = new WeakMap([[key1, 'AAA'], [key2, 23]])
weakMap.set(key3, '广东')

console.log(weakMap.has(key1)) // true
console.log(weakMap.get(key1)) // AAA
weakMap.delete(key1)
console.log(weakMap.has(key)) // false
```
`Map`集合和`Weak Map`集合有许多共同的特性，但它们之间还是有一定的差别的：
* `Weak Map`集合的键名必须为对象，添加非对象会报错。
* `Weak Map`集合不可迭代。
* `Weak Map`集合不支持`forEach`方法。
* `Weak Map`集合不支持`clear`方法。
* `Weak Map`集合不支持`size`属性。

## 迭代器(Iterator)和生成器(Generator)

### 循环语句的问题

### 什么是迭代器

### 什么是生成器

### 可迭代对象和for-of循环

### 内建迭代器

### 展开运算符和非数组可迭代对象

### 高级迭代器功能

### 异步任务执行

## JavaScript中的类

### ES5中的近类结构

### 类的声明

### 类的表达式

### 访问器属性

### 可计算成员名称

### 生成器方法

### 静态成员

### 继承与派生类

### 构造函数中的new.target

## 改进的数组功能

### 创建数组

### 数组新方法

### 定性数组

### 定性数组和普通数组的对比

## Promise和异步编程

### 异步编程的背景知识

### Promise基础

### 全局Promise拒绝处理

### 串联Promise

### 响应对个Promise

### 自Promise继承

### 基于Promise的异步任务执行

## 代理(Proxy)和反射(Reflect)API

### 数组问题

### 代理和反射

### 创建一个简单的代理

### 使用set陷阱

### 使用get陷阱

### 使用has陷阱

### 使用deleteProperty陷阱

### 使用原型代理陷阱

### 使用对象可扩展陷阱

### 使用属性描述符陷阱

### 使用ownKeys陷阱

### 使用apply和construce陷阱

### 可撤销代理

### 解决数组问题

### 将代理作为原型

## 用模块封装代码

### 什么是模块

### 导出的基本语法

### 导入的基本语法

### 导出和导入时重命名

### 模块的默认值

### 重新导出一个绑定

### 无绑定导入

### 加载模块


